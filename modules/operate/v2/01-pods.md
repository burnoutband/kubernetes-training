# Pods 

## Module Objectives

1. Deploy the sample app to Kubernetes

---

## Deploy the Sample App to Kubernetes

In this section, you will deploy the mysql database, `gceme` frontend and backend apps to Kubernetes. We will use Kubernetes manifest files to describe the environment that the `gceme` binary/Docker image will be deployed to. We will use the `gceme` Docker image that you built in a previous module.

1. First change directories to the sample-app.

    ```shell
    cd google-k8s-workshop-v2/sample-app
    ```

1. Create the manifest to deploy the `db` MySQL Pod. Save the following as `manifests/db.yaml`:

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: db
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: very-secret-password
        ports:
        - containerPort: 3306
          name: mysql
    ```

    > Note: `mysql` is a public image, available on hub.docker.com. If we do not specify a full registry url such as gcr.io it will check the default Docker Hub registry.

1. Deploy the `db` Pod to Kubernetes.

    ```shell
    kubectl apply -f manifests/db.yaml
    ```

1. List all Pods.

    ```shell
    kubectl get pod
    ```

    ```
    NAME      READY     STATUS    RESTARTS   AGE
    db        1/1       Running   0          17s
    ```

1. Find out the `db` Pod IP address.

    ```shell
    kubectl describe pod db | grep IP
    ```

    It is also useful to take a look at the full output of the `kubectl describe pod` command.

1. Get your uploaded container image from Google Container Registry. PROJECT_ID should be set automatically as part of exercise 1. This image will be used for our three tier application in our manifests.

    ```shell
    echo $PROJECT_ID
    export IMAGE=gcr.io/$PROJECT_ID/sample-k8s-app:1.0.0
    echo $IMAGE
    ```

1. Create the manifest for the `backend` application. Save it as `manifests/backend.yaml` and deploy it to Kubernetes using `kubectl apply` command.

    ```yaml
    kind: Pod
    apiVersion: v1
    metadata:
      name: backend
    spec:
      containers:
      - name: backend
        image: <REPLACE_WITH_YOUR_OWN_IMAGE>
        command: ["app", "-mode=backend", "-run-migrations", "-port=8080", "-db-host=<REPLACE_WITH_MYSQL_IP>", "-db-password=very-secret-password" ]
        ports:
        - name: backend
          containerPort: 8080
    ```

    > Important: Replace the image and the MySQL `db` pod's ip address.

1. Find out the `backend` Pod IP address just like how we did for the `db` Pod.

1. Create the manifest for the `frontend` application. Save it as `manifests/frontend.yaml` and deploy it to Kubernetes using `kubectl apply` command.

    ```yaml
    kind: Pod
    apiVersion: v1
    metadata:
      name: frontend
    spec:
      containers:
      - name: frontend
        image: <REPLACE_WITH_YOUR_OWN_IMAGE>
        command: ["app", "-mode=frontend", "-backend-service=http://<REPLACE_WITH_BACKEND_IP>:8080", "-port=80"]
        ports:
        - name: frontend
          containerPort: 80
    ```

    > Important: Replace the image and the `backend` ip address.

1. Find out the `frontend` pod IP address.

1. In the Cloud Console go to the 'Compute Engine' -> 'VM instances' page and `ssh` to any of the worker nodes. Pods have only cluster-internal IP addresses and are not available from the outside by default.

1. From the node try to connect to both the `backend` and `frontend` using curl.

    ```shell
    curl <backend-ip>:8080
    curl <frontend-ip>
    ```


