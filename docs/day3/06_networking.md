## Networking

### Exercise 1: Installing Kubernetes Addons 

1. Create a `simple-service.yaml`

1. Add the following deployment and service to the manifest file.
    ```
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: simpleservice
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: simpleservice
      template:
        metadata:
          labels:
            app: simpleservice
        spec:
          containers:
          - name: simpleservice
            image: mhausenblas/simpleservice:0.5.0
            ports:
            - containerPort: 9876
            env:
            - name: SIMPLE_SERVICE_VERSION
              value: "0.9"

    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: simpleservice-svc
    spec:
      ports:
        - port: 80
          targetPort: 9876
      selector:
        app: simpleservice
    ```
    
1. Create the service
    ```
    kubectl create -f simple-service.yaml
    ```
 
1. SSH to any of the nodes and examine generated iptables rules
    ```
    sudo iptables-save | grep simpleservice
    ```
    The output should look like this 
    ```
    -A KUBE-SEP-5FXC3Y3RDI3GY23F -s 100.96.2.4/32 -m comment --comment "default/simpleservice-svc:" -j KUBE-MARK-MASQ
    -A KUBE-SEP-5FXC3Y3RDI3GY23F -p tcp -m comment --comment "default/simpleservice-svc:" -m tcp -j DNAT --to-destination 100.96.2.4:9876
    -A KUBE-SEP-UL7W7MQRDB5MWDF3 -s 100.96.2.3/32 -m comment --comment "default/simpleservice-svc:" -j KUBE-MARK-MASQ
    -A KUBE-SEP-UL7W7MQRDB5MWDF3 -p tcp -m comment --comment "default/simpleservice-svc:" -m tcp -j DNAT --to-destination 100.96.2.3:9876
    
    -A KUBE-SERVICES ! -s 100.96.0.0/11 -d 100.67.234.205/32 -p tcp -m comment --comment "default/simpleservice-svc: cluster IP" -m tcp --dport 80 -j KUBE-MARK-MASQ
    -A KUBE-SERVICES -d 100.67.234.205/32 -p tcp -m comment --comment "default/simpleservice-svc: cluster IP" -m tcp --dport 80 -j KUBE-SVC-6GA5KCFGZYCRFQCY
    -A KUBE-SVC-6GA5KCFGZYCRFQCY -m comment --comment "default/simpleservice-svc:" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-UL7W7MQRDB5MWDF3
    -A KUBE-SVC-6GA5KCFGZYCRFQCY -m comment --comment "default/simpleservice-svc:" -j KUBE-SEP-5FXC3Y3RDI3GY23F
    ```

    Now let's try to understand this rules. The first 4 rules are responsible for setting up NAT on the top of the pod network.

    * `-A KUBE-SEP-5FXC3Y3RDI3GY23F -s 100.96.2.4/32 -m comment --comment "default/simpleservice-svc:" -j KUBE-MARK-MASQ` - `100.96.2.4` is the IP of one of the pods. All trafic that comes from this IP addres is redirected to `KUBE-MARK-MASQ` target, which [masquerades](http://www.syrlug.org/contrib/ipmasq.html) the trafic. 
    * `-A KUBE-SEP-5FXC3Y3RDI3GY23F -p tcp -m comment --comment "default/simpleservice-svc:" -m tcp -j DNAT --to-destination 100.96.2.4:9876` - here DNAT is used to publish the service from internal (pod) network, using [DNAT](http://linux-ip.net/html/nat-dnat.html)
    * The third and forth rules are exactly the same, but for the second pod.
    * `-A KUBE-SERVICES ! -s 100.96.0.0/11 -d 100.67.234.205/32 -p tcp -m comment --comment "default/simpleservice-svc: cluster IP" -m tcp --dport 80 -j KUBE-MARK-MASQ` - `100.67.234.205` is the IP address of the service. 
    

1. Cleanup
    ```
    kubectl delete -f simple-service.yaml
    ```
