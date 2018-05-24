## Logging

Setting up an EFK stack on Kubernetes.

### Exercise 1: Installing the Kubernetes elasticsearch logging addon

1. Install the [Elasticsearch Logging]() addon.
    ```
    kubectl create -f https://raw.githubusercontent.com/kubernetes/kops/master/addons/logging-elasticsearch/v1.6.0.yaml
    ```

    This will add Fluentd for parsing/shipping, Elasticsearch for storing/indexing and Kibana for visualization.

1. Start a proxy on port 8080.
    ```
    kubectl proxy -p 8080
    ```
    
1. Forward port 8080 from the Cloud Shell to your local machine. 

    From Cloud Shell top bar select option `Preview on port 8080` 

1. View the Kibana service on the /proxy/ endpoint.

    Append `api/v1/namespaces/kube-system/services/kibana-logging/proxy` to the top level domain.
    
    E.g., `https://8080-dot-3438793-dot-devshell.appspot.com/api/v1/namespaces/kube-system/services/kibana-logging/proxy`

1. Explore the Kibana Dashboard and what information you can find there.

### Exercise 2: (Optional)