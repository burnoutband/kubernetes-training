# Installing a Kubernetes Cluster on Google Cloud Platform (GCP) with Kubernetes Operations (kops)

## Prerequisites

* Install Kubernetes CLI [kubctl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
* Install Google Cloud SDK [gcloud tools](https://cloud.google.com/sdk/docs/)
* Install Kubernetes Operations [kops](https://github.com/kubernetes/kops/blob/master/docs/install.md)

>Note: We can use the Google Cloud Shell which already has kubectl and gcloud tools installed.

### Step 1: Accessing GCP and creating a project

1. Launch the GCP console from your browser. [GCP Console](https://console.cloud.google.com/)

2. Login and select the assigned project or create a new one from the top menu.

3. Enable the Compute Engine API here [GCP Console](https://console.cloud.google.com/apis/api/compute.googleapis.com/)

### Step 2: Configure the gcloud SDK in your shell

1. Run the following command, select your project and us-west1-b for compute zone

```
glcoud init
```

2. Set default credentials with the following command

```
gcloud auth application-default login
```

### Step 3: Creating a state store

1. Run the following command

    ```
    gsutil mb gs://kubernetes-cluster/
    ```
    Here we are creating a GCP bucket to store the cluster configuration for kops

### Step 4: Create the cluster configuration

1. Run the following commands

   ```
   PROJECT=`gcloud config get-value project`
   export KOPS_FEATURE_FLAGS=AlphaAllowGCE # to unlock the GCE features
   kops create cluster simple.k8s.local --zones us-west1-b --state gs://kubernetes-cluster/ --project=${PROJECT}
   ```
   >Note: This only created the configuration which can be viewed here:
   ```
   kops get cluster --state gs://kubernetes-clusters/
   kops get cluster --state gs://kubernetes-clusters/ simple.k8s.local -oyaml
   kops get instancegroup --state gs://kubernetes-clusters/ --name simple.k8s.local
   ```
   We can set a variable for the state store to make commands shorter:
   ```
   export KOPS_STATE_STORE=gs://kubernetes-clusters/
   ```
### Step 5: Build the cluster in GCE

1. Run the following command and confirm the output

   ```
   kops update cluster simple.k8s.local
   ```
   To proceed with the operation we need to confirm the command by running
   ```
   kops update cluster simple.k8s.local --yes
   ```
   After a few minutes the cluster will be ready and can be viewed from:
   ```
   kubectl get nodes
   ```