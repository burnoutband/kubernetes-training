## Service Catalog

The service catalog provisions instances of services that can be consumed by applications in a standardised way using the [Open Service Broker API specification](https://www.openservicebrokerapi.org/).

Cluster operations can create a service instance for any service brokers that are registered with the k8s cluster. These service instances can then be bound to applications running inside the k8s cluster. For example, Service Catalog can connect to the Google Cloud Platform (GCP) [Service Broker](https://cloud.google.com/kubernetes-engine/docs/concepts/google-cloud-platform-service-broker) and easily provision GCP services such as BigQuery, BigTable, Pub/Sub, Cloud Storage, Cloud IAM etc. This reduces the need for cluster operators to build and manage their own services and utilize managed services.

Service brokers are a way of connecting to external services without having to manually import information such as credentials or endpoints. External service dependencies are modeled as Kubernetes resources, which can easily be interested into existing container deployment processes.

Service catalog can be installed with either [Helm](https://kubernetes.io/docs/tasks/service-catalog/install-service-catalog-using-helm/) or using the [SC tool](https://kubernetes.io/docs/tasks/service-catalog/install-service-catalog-using-sc/).

### Exercise 1: Install Service Catalog with Helm

1. Add the service catalog helm repository
    ```
    helm repo add svc-cat https://svc-catalog-charts.storage.googleapis.com
    ```

1. Find the service catalog helm chart
    ```
    helm search service-catalog
    ```

1. Configure tiller to have cluster-admin access if it doesn't already
    ```
    kubectl create clusterrolebinding tiller-cluster-admin --clusterrole=cluster-admin --serviceaccount=kube-system:default
    ```

1. Install the service catalog to the catalog namespace
    ```
    helm install svc-cat/catalog --name catalog --namespace catalog
    ```

### Exercise 2: Deploy the GCP service broker

1. Install the sc CLI which is written in go. Google Cloud Shell already has go pre-installed.
    ```
    go get -u github.com/GoogleCloudPlatform/k8s-service-catalog/installer/cmd/sc
    ```

1. Install the sc CLI which is written in go. Google Cloud Shell already has go pre-installed.
    ```
    sc add-gcp-broker
    ```

1. Verify the service broker is available and ready
    ```
    kubectl get clusterservicebrokers -o 'custom-columns=BROKER:.metadata.name,STATUS:.status.conditions[0].reason'
    ```

1. Allow the service broker to access and create GCP resources
    ```
    GCP_PROJECT_ID=$(gcloud config get-value project)
    GCP_PROJECT_NUMBER=$(gcloud projects describe $GCP_PROJECT_ID --format='value(projectNumber)')
    gcloud projects add-iam-policy-binding ${GCP_PROJECT_ID} --member serviceAccount:${GCP_PROJECT_NUMBER}@cloudservices.gserviceaccount.com --role=roles/owner
    ```

### Exercise 3: Use svcat CLI to view broker classes and plans

1. Install the svcat CLI for interacting with the Kubernetes service catalog
    ```
    curl -sLO https://download.svcat.sh/cli/latest/linux/amd64/svcat
    chmod +x ./svcat
    mv ./svcat $HOME/bin/
    svcat version --client
    ```

1. Create a GCP IAM Service account
    ```
    svcat provision gcp-iam \
    --class cloud-iam-service-account \
    --plan beta \
    --namespace default \
    --param accountId=gcp-sa
    ```

1. Check that the service account was created successfully
    ```
    svcat get instance gcp-iam
    ```

1. List available GCP services
    ```
    svcat get classes
    ```

1. List available GCP service plans
    ```
    svcat get plans
    ```
