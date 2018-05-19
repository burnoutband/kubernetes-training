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

1. Create empty `ingress-sample-apps.yaml` file

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
    ---
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
          - path: /sample-app2
            backend:
              serviceName: app2-svc
              servicePort: 80
    ```

1. Deploy everything
    ```
    kubectl apply -f ingress-sample-apps.yaml
    ```

1. Run the following command to get ingress IP address
    ```
    kubectl get ing sample-app-ingress
    ```

1. Open the following 2 urls `<ingress-ip>/sample-app1` and `<ingress-ip>/sample-app2`. Make sure that they lead to app1 and app2 respectedly.

### Exercise 2 (Optional): Specify app host

1. Now app1 and app2 should be accessed using different dns names.
1. Modify you `/etc/hosts` and set `app1.com` and `app2.com` domains to be resolved to the ingress IP address.
1. Modify ingress defenition apropriately. Find section `Name based virtual hosting` in [this](https://kubernetes.io/docs/concepts/services-networking/ingress/#types-of-ingress) document for reference.
1. Access `app1.com` and `app2.com` from your web browser.

### Exercise 3 (Optional): Use TLS

1. Create 2 self-signed certificates for `app1`  and `app2` [link](https://stackoverflow.com/questions/10175812/how-to-create-a-self-signed-certificate-with-openssl?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)
1. Create 2 secrets for `app1`  and `app2`. Each secret should contain corresponding certificate and private key.
1. Add `tls` section to the ingress definition. Use `TLS` section in [this](https://kubernetes.io/docs/concepts/services-networking/ingress/#types-of-ingress) document for reference.
1. Redeploy ingress, open each app in a web browser and examine certificate details. Make sure that each app now uses its own certificates. Use [this](https://www.ssl2buy.com/wiki/how-to-view-ssl-certificate-details-on-chrome-56) link to see how a certificate can be viewed in chrome.

