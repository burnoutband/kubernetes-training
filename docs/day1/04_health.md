## Health Checks

### Exercise 1: Deploy a pod with a health check 

1. Create a pod that exposes an endpoint /health, responding with an HTTP 200 status code
    Save the following file as `hc.yml`
    ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: hc
    spec:
      containers:
      - name: simpleservice
        image: mhausenblas/simpleservice:0.5.0
        ports:
        - containerPort: 9876
        livenessProbe:
          initialDelaySeconds: 2
          periodSeconds: 5
          httpGet:
            path: /health
            port: 9876
    ```
    Pay atension to `livenessProbe` section.

1. Deploy the pod
    ```
    kubectl create -f hc.yml
    ```

1. Describe the pod; it should be considered healthy
    ```
    kubectl describe pod hc
    ```

1. Now we launch a bad pod, that is, a pod that has a container that randomly (in the time range 1 to 4 sec) does not return a 200 code
    Save the following file as `bad-hc.yml`
    ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: badpod
    spec:
      containers:
      - name: simpleservice
        image: mhausenblas/simpleservice:0.5.0
        ports:
        - containerPort: 9876
        env:
        - name: HEALTH_MIN
          value: "1000"
        - name: HEALTH_MAX
          value: "4000"
        livenessProbe:
          initialDelaySeconds: 2
          periodSeconds: 5
          httpGet:
            path: /health
            port: 9876
    ```

1. Deploy the pod
    ```
    kubectl create -f bad-hc.yml
    ```

1. check events at the bad pod
    ```
    kubectl describe pod badpod
    ```
    You should see that bad pod was restarted several times.

### Exercise 2: Use readiness probe 

1. Create a pod `readiness.yml` with a readinessProbe that kicks in after 10 seconds
    ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: ready
    spec:
      containers:
      - name: simpleservice
        image: mhausenblas/simpleservice:0.5.0
        ports:
        - containerPort: 9876
        readinessProbe:
          initialDelaySeconds: 10
          httpGet:
            path: /health
            port: 9876
    ```

1. Deploy the pod
    ```
    kubectl create -f readiness.yml
    ```

1. Looking at the events of the pod. 
    You should see that, eventually, the pod is ready to serve traffic
```
kubectl describe pod ready
```

### Exercise 3 (Optional): Create health check for nginx pod 

1. Deploy a pod that runs nginx and uses port 80 and root path for the health check. Ensure that pod is healthy.
1. Change health check configuration to make pod unhealthy.
1. Observe whether kubernetes tries to restart the unhealthy pod.

