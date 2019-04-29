# Services

## Module Objectives

1. Use Services and service discovery
1. Use LoadBalancer services

---
## Use Services and Service Discovery

In the previous exercise, we manually copied IP addresses to connect Pods. This is not only inconvenient, but error prone. After a Pod is restarted its IP address changes, this will prevent your Pods from reconnecting when failures occur. In our current setup it is impossible to load balance the traffic between multiple instances of the `backend` Pod. Services will help us to fix all of these issues.

1. Add a label to the `db` Pod. In the `manifests/db.yaml` file update the `metadata` section. The whole section should look like the following:

    ```yaml
    metadata:
      name: db
      labels:
        app: gceme
        role: db
    ```

    Use `kubectl apply` to apply the changes. Here we are adding labels to the `db` Pod. Services use labels under the hood to discover which Pods to direct traffic to.

1. Create a `db` service. Create `manifests/db-svc.yaml` with the following content:

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: db
    spec:
      type: ClusterIP
      ports:
        - port: 3306
      selector:
        app: gceme
        role: db
    ```

1. Apply the `db` Service.

    ```shell
    kubectl apply -f manifests/db-svc.yaml
    ```

1. Add a label to the `backend` Pod:

    ```yaml
    metadata:
      name: backend
      labels:
        app: gceme
        role: backend
    ```

1. Update the `backend` Pod startup command.

    ```yaml
    command: ["app", "-mode=backend", "-run-migrations", "-port=8080", "-db-host=db", "-db-password=very-secret-password" ]
    ```

1. Delete and create the `backend`. Note that `kubectl apply` will not work for us this time, because we are updating the Pod startup command.

    Use `kubectl delete pod backend` to delete the Pod.

    Ensure the pod is terminated with `kubectl get pods` before you re-apply `kubectl apply -f manifests/backend.yaml`, sometimes Pods will drain or take time to close db connections.

    > Note: You can alternatively delete using the file, e.g. `kubectl delete -f manifests/backend.yaml`.

    > Tip: You can quickly delete many Pods by specifying the folder or wildcard, e.g. `kubectl delete -f manifests/` `kubectl delete -f *.yaml`.

1. Create a `backend` Service file `manifests/backend-svc.yaml`.

    ```yaml
    kind: Service
    apiVersion: v1
    metadata:
      name: backend
    spec:
      ports:
      - name: http
        port: 8080
        targetPort: 8080
        protocol: TCP
      selector:
        role: backend
        app: gceme
    ```

1. Apply the `backend` Service.

    ```shell
    kubectl apply -f manifests/db-svc.yaml
    ```

1. Update the `frontend` startup command.

    ```yaml
    command: ["app", "-mode=frontend", "-backend-service=http://backend:8080", "-port=80"]
    ```

1. Delete and recreate the `frontend` Pod.

1. SSH to a worker node and check that the app is working.

## Use LoadBalancer Services

Next thing we have to do is to expose the app to the external world. We can use the Service type `LoadBalancer` in order to do that.

1. Create a `frontend` load balancer Service. Save the following file as `manifests/frontend-svc.yaml` and use `kubectl apply` command to create the frontend service.

    ```yaml
    kind: Service
    apiVersion: v1
    metadata:
      name: frontend
    spec:
      type: LoadBalancer
      ports:
      - name: http
        port: 80
        targetPort: 80
        protocol: TCP
      selector:
        app: gceme
        role: frontend
    ```

1. Add a label to the `frontend` and apply changes.

    ```yaml
    metadata:
      name: frontend
      labels:
        app: gceme
        role: frontend
    ```

1.  Run `kubectl get services` to list all Services.

1. Retrieve the External IP for the `frontend` Service (this can take a few minutes to provision on the IaaS level):

    ```shell
    kubectl get service frontend
    ```

    ```
    NAME             TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)        AGE
    frontend         LoadBalancer   10.35.254.91   35.196.48.78   80:31088/TCP   1m
    ```

    > Note: This may take a few minutes to appear as the load balancer is being provisioned

1. Copy the external IP and open it in your browser.

    > Note: Make sure that the application is working correctly.

1. In GCP Cloud Console, find and investigate the external IP address that the `LoadBalancer` service type created
    * VPC Network -> External IP addresses


## Optional Exercises

### Blue green deployment

1. Assign label "app=blue" to the frontend pod.
1. Modify frontend service selector to use the label "app=blue"  
1. Deploy a second frontend with label "app=green". The second pod should contain the same application. (in a real scenario this should be a different version of the app, but for your example, you can use exactly the same app)
1. [Exec](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/) inside the second pod and modify [this](../../../../sample-app/templates/base.html) file. Replace all occurrences of the word "blue" by "orange" (this should change interface color) 
1. Change service selector to "app=green" and make sure that now the service switched to the second pod.
