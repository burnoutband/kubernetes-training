## Services

A service is an abstraction for pods, providing a stable, virtual IP (VIP) address.

While pods may come and go, services allow clients to reliably connect to the containers running in the pods, using the VIP. The virtual in VIP means it’s not an actual IP address connected to a network interface but its purpose is purely to forward traffic to one or more pods.

Keeping the mapping between the VIP and the pods up-to-date is the job of kube-proxy, a process that runs on every node, which queries the API server to learn about new services in the cluster.

### Exercise 1: Deploying PHP Guestbook application with Redis

1. Save the following file as `redis-master-deployment.yaml`
    ```
    apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
    kind: Deployment
    metadata:
      name: redis-master
    spec:
      selector:
        matchLabels:
          app: redis
          role: master
          tier: backend
      replicas: 1
      template:
        metadata:
          labels:
            app: redis
            role: master
            tier: backend
        spec:
          containers:
          - name: master
            image: redis
            resources:
              requests:
                cpu: 100m
                memory: 100Mi
            ports:
            - containerPort: 6379
    ```

1. Create the Redis Master Deployment
    ```
    kubectl apply -f redis-master-deployment.yaml
    ```

1. Query the list of Pods to verify that the Redis Master Pod is running.
    ```
    kubectl get pods
    ```

1. Save the following file as `redis-master-service.yaml` 
    ```
    apiVersion: v1
    kind: Service
    metadata:
      name: redis-master
      labels:
        app: redis
        role: master
        tier: backend
    spec:
      ports:
      - port: 6379
        targetPort: 6379
      selector:
        app: redis
        role: master
        tier: backend
    ```
    Pay attension to `selector` and `ports` fields. Make sure you understand how service is connected to deployment. 

1. Deploy the service.
    ```
    kubectl apply -f redis-master-service.yaml
    ```

1. Save the following file as `redis-slave-deployment.yaml`
    ```
    apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
    kind: Deployment
    metadata:
      name: redis-slave
    spec:
      selector:
        matchLabels:
          app: redis
          role: slave
          tier: backend
      replicas: 2
      template:
        metadata:
          labels:
            app: redis
            role: slave
            tier: backend
        spec:
          containers:
          - name: slave
            image: gcr.io/google_samples/gb-redisslave:v1
            resources:
              requests:
                cpu: 100m
                memory: 100Mi
            env:
            - name: GET_HOSTS_FROM
              value: dns
            ports:
            - containerPort: 6379
    ```

1. Apply the redis slave deployment.
    ```
    kubectl apply -f redis-slave-deployment.yaml
    ```

1. Save the following file as `redis-slave-service.yaml`
    ```
    apiVersion: v1
    kind: Service
    metadata:
      name: redis-slave
      labels:
        app: redis
        role: slave
        tier: backend
    spec:
      ports:
      - port: 6379
      selector:
        app: redis
        role: slave
        tier: backend
    ```

1. Deploy the redis slave service.
    ```
    kubectl apply -f redis-slave-service.yaml
    ``` 

1. Save the following file as `frontend-deployment.yaml`
    ```
    apiVersion: apps/v1 
    kind: Deployment
    metadata:
      name: frontend
    spec:
      selector:
        matchLabels:
          app: guestbook
          tier: frontend
      replicas: 3
      template:
        metadata:
          labels:
            app: guestbook
            tier: frontend
        spec:
          containers:
          - name: php-redis
            image: gcr.io/google-samples/gb-frontend:v4
            resources:
              requests:
                cpu: 100m
                memory: 100Mi
            env:
            - name: GET_HOSTS_FROM
              value: dns
            ports:
            - containerPort: 80
    ```

1. Apply the frontend deployment. 
    ```
    kubectl apply -f frontend-deployment.yaml
    ```

1. Save the following file as `frontend-service.yaml`
    ```
    apiVersion: v1
    kind: Service
    metadata:
      name: frontend
      labels:
        app: guestbook
        tier: frontend
    spec:
      type: LoadBalancer
      ports:
      - port: 80
      selector:
        app: guestbook
        tier: frontend
    ```

1. Deploy the fronted service.
    ```
    kubectl apply -f frontend-service.yaml

    ```

1. Run `kubectl get services` to list all services.

1. Run the following command to get the IP address for the frontend Service.
    ```
    kubectl get service frontend
    ```

1. Copy the External IP address, and load the page in your browser to view the application.

1. Run the following command to scale up the number of frontend Pods
    ```
    kubectl scale deployment frontend –replicas=5

    ```

