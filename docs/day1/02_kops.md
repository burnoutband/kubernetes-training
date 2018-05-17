# Installing a Kubernetes Cluster on Google Cloud Platform (GCP) with Kubernetes Operations (kops)

## Prerequisites

* Kubernetes CLI [kubctl documentation](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
* Google Cloud SDK [gcloud tools documentation](https://cloud.google.com/sdk/docs/)
* Kubernetes Operations [kops documentation](https://github.com/kubernetes/kops/blob/master/docs/install.md)

>Note: We will use the Google Cloud Shell which already has kubectl and gcloud tools installed.

### Step 1: Accessing GCP

1. Launch the GCP console from your browser. [GCP Console](https://console.cloud.google.com/)

2. Login and select the assigned project or create a new one from the top menu.

3. Enable (if not aleady) the Compute Engine API here [GCP Console](https://console.cloud.google.com/apis/api/compute.googleapis.com/)

### Step 2: Launching the Google Cloud Shell

1. From the top right menu of the [GCP Console](https://console.cloud.google.com/) launch the shell.

2. Run the following commands to set the proper region and zone:

   ```console
   gcloud config set compute/region us-west1

   gcloud config set compute/zone us-west1-c
   ```
>Note: Further commands will all be run in this shell.

### Step 3: Installing kops

1.  Run the following commands to download and install kops:

   ```console
   curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64

   chmod +x kops-linux-amd64
   
   sudo mv kops-linux-amd64 /usr/local/bin/kops
   ```

### Step 4: Creating a state store

1. Run the following command and replace `<unique value>` with your username or other.

    ```console
    gsutil mb gs://kubernetes-<unique value>/
    ```
    Here we are creating a GCP bucket to store the cluster configuration for kops

### Step 5: Creating the cluster configuration

1. Run the following commands - remember to replace `<unique value>` with what you used in the previous step.

   ```console
   PROJECT=`gcloud config get-value project`
   
   export KOPS_FEATURE_FLAGS=AlphaAllowGCE # to unlock the GCE features
   
   kops create cluster simple.k8s.local --zones us-west1-b --state gs://kubernetes-<unique value>/ --project=${PROJECT}
   ```
   
   >Note: This only created the configuration which can be viewed here:
   
   ```console
   kops get cluster --state gs://kubernetes-<unique value>/
   
   kops get instancegroup --state gs://kubernetes-<unique value>/ --name simple.k8s.local
   
   kops get cluster --state gs://kubernetes-<unique value>/ simple.k8s.local -oyaml
   ```
   >Note: We can set a variable for the state store to make commands shorter:

   ```console
   export KOPS_STATE_STORE=gs://kubernetes-<unique value>/
   ```

### Step 6: Building the cluster in GCE

1. Run the following command and confirm the output

   ```console
   kops update cluster simple.k8s.local
   ```
   
   To proceed with the operation we need to confirm the command by running
   
   ```console
   kops update cluster simple.k8s.local --yes
   ```
   
   After a few minutes the cluster will be ready and can be viewed from kubectl
   
   ```console
   kubectl cluster-info
   
   kubectl get nodes
   ```