## Ingress 

An API object that manages external access to the services in a cluster, typically HTTP.

Ingress can provide load balancing, SSL termination and name-based virtual hosting.


### Exercise 1: Deploy sample app using ingress 

1. Deploy NGINX ingress controller
    ```
    kubectl create -f https://raw.githubusercontent.com/kubernetes/kops/master/addons/ingress-nginx/v1.6.0-gce.yaml
    ```
    This command deployes the controller in a separete namespace. The controller is just a set of different kubernetes objects: pods, services, etc. The controller is responsible for hosting nginx inside a pod and reconfiguring it whenever new ingress is deployed. Don't forget to open controller definition and see what is inside.

1. Use the following command to ensure that the controller is deployed corectly
    ```
    kubectl --namespace kube-ingress get pods
    ```
    The output should be like this
    ```
    NAME                                     READY     STATUS    RESTARTS   AGE
    ingress-nginx-785fb9fcc5-vdxq9           1/1       Running   3          2h
    nginx-default-backend-6f675f4c45-rfl8n   1/1       Running   0          2h
    ```

1. Create empty `ingess-sample-apps.yaml` file

1. Add app1 deployment to  `ingess-sample-apps.yaml`

    ```
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      name: app1
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: app1
      template:
        metadata:
          labels:
            app: app1
        spec:
          containers:
          - name: app1
            image: nginxdemos/hello:plain-text
            ports:
            - containerPort: 80
    ```

1. Add app2 deployment to  

    ```
    ---
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      name: app2
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: app2 
      template:
        metadata:
          labels:
            app: app2 
        spec:
          containers:
          - name: app2 
            image: nginxdemos/hello:plain-text
            ports:
            - containerPort: 80
    ```

1. Add app1 service

    ```
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: app1-svc
    spec:
      ports:
      - port: 80
        targetPort: 80
        protocol: TCP
        name: http
      selector:
        app: app1
    ```

1. Add app2 service

    ```
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: app2-svc
    spec:
      ports:
      - port: 80
        targetPort: 80
        protocol: TCP
        name: http
      selector:
        app: app2
    ```

1. Add ingress definition

    ```
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: sample-app-ingress
    spec:
      rules:
      - http:
          paths:
          - path: /sample-app1
            backend:
              serviceName: app1-svc
              servicePort: 80
          - path: /sample-app1
            backend:
              serviceName: app2-svc
              servicePort: 80
    ```

1. Deploy everything
    ```
    kubectl apply -f ingess-sample-apps.yaml
    ```
