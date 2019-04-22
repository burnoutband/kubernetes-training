# Deployments 

## Module Objectives

- Convert a pod to a deployment
- Scale/update/rollout/rollback the deployment

---

## Convert a Pod to a Deployment

There are a couple of problems with our current setup.

* If the application inside a Pod crashes, nobody will restart the Pod.
* We already use Services to connect our Pods to each other, but we don't have an efficient way to scale the Pods behind a Service and load balance traffic between Pod instances.
* We don't have an efficient way to update our Pods without a downtime. We also can't easily rollback to the previous version if we need to do so.

    **Deployments** can help to eliminate all these issues. Let's convert all our 3 Pods to Deployments.

1. Delete all running Pods.

    ```shell
    kubectl delete pod --all
    ```

1. Open the `manifests/db.yaml` file.

1. Delete the **top 2 lines**.

    ```yaml
    apiVersion: v1
    kind: Pod
    ```

1. **Indent/add 4 spaces** in the beginning of each line. If you use vim for editing you can check [this](http://vim.wikia.com/wiki/Shifting_blocks_visually) link to learn how to easily shift blocks of text in vim.

    > Tip: You can configure VIM to indent spaces with Tab using the following command `vim :set ts=4 sw=4 sts=4 et :`

1. After indenting, add the following block to the **top** of the file:

    ```yaml
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      name: db
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: gceme
          role: db
      template:
    ```

    Your Pod definition becomes a template for the newly created Deployment. The Deployment Controller handles creating a ReplicaSet and creating the Pods.

1. **Repeat these steps** for the `frontend` and `backend` Pods. Don't forget to change the name of the Deployment and the `machLabels` element.

1. List Deployments.

    ```shell
    kubectl get deployments
    ```

    ```
    NAME       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    backend    1         1         1            0           1m
    db         1         1         1            1           1m
    frontend   1         1         1            1           6s
    ```

1. List ReplicaSets.

    ```shell
    kubectl get rs
    ```

    ```
    NAME                  DESIRED   CURRENT   READY     AGE
    backend-5c46cc4bb9    1         1         1         12m
    db-77df47c5dd         1         1         1         12m
    frontend-654b5ff445   1         1         1         11m
    ```

1. List Pods.

    ```shell
    kubectl get pod
    ```

    ```
    NAME                        READY     STATUS    RESTARTS   AGE
    backend-5c46cc4bb9-wf7qj    1/1       Running   0          12m
    db-77df47c5dd-ssgs6         1/1       Running   0          12m
    frontend-654b5ff445-b6fgs   1/1       Running   0          11m
    ```

1. List Services and connect to the `frontend` LoadBalancer service's IP in a web browser. Verify that the app is working fine.

    ```shell
    kubectl get services
    ```

## Scale/Update/Rollout/Rollback the Deployment

    ```shell
    kubectl get services
    ```

### Scale the Deployment

1. Edit `manifests/backend.yaml`. Update the number of replicas to 3 and apply changes.

1. In your browser refresh the application several times. Notice that `Container IP` field sometimes changes. This indicates that the request comes to a different backend Pod.

### Update and Rollout the Deployment

1. Edit `main.go` file. Change `version` to `1.0.1` and save the file. (Around line 60)

1. Rebuild the container image with a new tag.

    ```shell
    export IMAGE=gcr.io/$PROJECT_ID/sample-k8s-app:1.0.1
    docker build . -t $IMAGE
    docker push $IMAGE
    ```

1. Update `manifests/backend.yaml` to use the new version of the container image and apply the changes.

    Replace the image version in both the `initContainers` and `containers` sections.

    ```yaml
    image: gcr.io/<PROJECT-ID>/sample-k8s-app:1.0.1
    ```

    > Note: Remember to replace `<PROJECT-ID>` with your project Id.

1. Watch how the Pods are rolled out in real time.

    ```shell
    watch kubectl get pod
    ```

1. Open the application in the browser and make sure the `backend` version is updated.

1. View the Deployment rollout history.

    ```shell
    kubectl rollout history deployment/backend
    ```

    ```
    deployments "backend"
    REVISION  CHANGE-CAUSE
    1         <none>
    2         <none>
    ```

### Rollback the Deployment

1. Rollback to the previous revision

    ```shell
    kubectl rollout undo deployment/backend
    ```

1. Make sure the application now shows version `1.0.0` instead of `1.0.1`.

## Optional Exercises

### Configure rollover parameters

Set [Max Unavailable](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#max-unavailable) and [Max Surge](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#max-surge) parameters to non default values. observe what changes in the rollower process.
