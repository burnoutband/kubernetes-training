## Create federated cluster

1. Create a DNS zone for your clusters in GCP (forward it to GCP if needed). For example `soroko-gcp.altoros.com`.

1. Deploy 3 clusters:
    * `PROJECT=$(gcloud config get-value project)`
    * `kops create cluster first.soroko-cf.altoros.com --zones us-west1-c --project=${PROJECT}`
    * `kops create cluster second.soroko-gcp.altoros.com --zones us-west1-b --project=${PROJECT}`
    * `kops create cluster third.soroko-gcp.altoros.com --zones us-west1-a --project=${PROJECT}`

(Note: due to a [bug](https://github.com/kubernetes/kops/issues/5281) you need to edit nodes/master definitions to use `cos-cloud/cos-stable-60-9592-90-0` image before you start deploy. To do that see instructions from `kops` after you run `create` command (look for `kops edit ...` commands).)

1. Check that `kops` created DNS records for you in `soroko-gcp.altoros.com` zone

1. [**HACK**](https://github.com/kubernetes/kops/issues/5295): you need to add DNS permissions to your nodes:
   * Open `Instance templates` in GCP and find template for your first cluster nodes (e.g. `nodes-first-soroko-gcp-altoros-com-1528384498`)
   * Edit it and change `Cloud API access scopes` to `Allow full access to all Cloud APIs`
   * Save template
   * Go to `Instance groups` and find group for your first cluster nodes (e.g. `c-nodes-first-soroko-gcp-altoros-com`)
   * Choose `Rolling update`, choose new template, click `Update`

1. Choose a DNS zone for your federation, for example `kubefed.soroko-gcp.altoros.com`. This zone must be controlled via GCP `Cloud DNS`. In our case we create new zone `kubefed.soroko-gcp.altoros.com` and then the `NS` record for this `kubefed.soroko-gcp.altoros.com` in `soroko-gcp.altoros.com` zone (use `NS` servers from `kubefed.soroko-gcp.altoros.com` zone when create `NS` record in `soroko-gcp.altoros.com` zone).

1. Install `kubefed`

1. Validate clusters, e.g. for each cluster do:
    * `kops export kubecfg <name>.soroko-gcp.altoros.com`
    * `kops validate cluster`

1. Deploy the federation control plane (note the `.` at the end of the DNS name): `kubefed init fellowship --host-cluster-context=first.soroko-gcp.altoros.com --dns-provider="google-clouddns" --dns-zone-name="kubefed.soroko-gcp.altoros.com."`

1. Check new resources:
    * `kubectl get all -n federation-system --context=first.soroko-gcp.altoros.com`

1. Check `controller-manager` pod:
    * `kubectl config use-context first.soroko-gcp.altoros.com`
    * `kubectl -n federation-system get pods`
    * `kubectl -n federation-system get pod controller-manager-85f8bb8bf-ggbb2`
    * `kubectl -n federation-system describe pod controller-manager-85f8bb8bf-ggbb2`
    * `kubectl -n federation-system logs controller-manager-85f8bb8bf-ggbb2 -f`

1. Check that `controller-manager` created DNS records for you in `kubefed.soroko-gcp.altoros.com` zone

1. Create default namespace: `kubectl get namespace --context=fellowship` and `kubectl create namespace default --context=fellowship`

1. Use federation context: `kubectl config use-context fellowship`

1. Join clusters:
    * `kubefed join second --host-cluster-context=first.soroko-gcp.altoros.com --cluster-context=second.soroko-gcp.altoros.com`
    * `kubefed join third --host-cluster-context=first.soroko-gcp.altoros.com --cluster-context=third.soroko-gcp.altoros.com`

1. List clusters: `kubectl get clusters`

1. Deploy `deployment` and `service`:
    * `kubectl config use-context fellowship`
    * `wget https://raw.githubusercontent.com/madeden/blogposts/master/k8s-federation/src/manifests/microbots-deployment.yaml`
    * `wget https://raw.githubusercontent.com/madeden/blogposts/master/k8s-federation/src/manifests/microbots-svc.yaml`
    * Edit `microbots-svc.yaml` to use `LoadBalancer`
    * Edit `microbots-deployment.yaml` to use 2 replicas
    * Deploy `deployment`: `kubectl apply -f microbots-deployment.yaml`
    * Deploy `service`: `kubectl apply -f microbots-svc.yaml`
    * Check created resources: `kubectl get all --all-namespaces`

1. Check how resources were distributed across clusters:
    * `kubectl get all --context=second.soroko-gcp.altoros.com`
    * `kubectl get all --context=third.soroko-gcp.altoros.com`

1. Check new DNS records in `kubefed.soroko-gcp.altoros.com` zone

1. Open deployed service via `LoadBalancer` IPs and DNS name.

1. Run something ad-hoc, e.g.:
    * `kubectl --context=fellowship run simpleservice --image=mhausenblas/simpleservice:0.5.0 --port=9876 -r 2`
    * `kubectl --context=fellowship expose deployment/simpleservice --type=LoadBalancer --port=80 --target-port=9876`

1. Check how resources were distributed across clusters:
    * `kubectl get all --context=fellowship`
    * `kubectl get all --context=second.soroko-gcp.altoros.com`
    * `kubectl get all --context=third.soroko-gcp.altoros.com`

1. Check new DNS records in `kubefed.soroko-gcp.altoros.com` zone

1. Open exposed service via `LoadBalancer` IPs and DNS name.

## Clean up

1. Remove all cluster from the federation:
    * `kubefed unjoin second --host-cluster-context=first.soroko-gcp.altoros.com --cluster-context=second.soroko-gcp.altoros.com`
    * `kubefed unjoin third --host-cluster-context=first.soroko-gcp.altoros.com --cluster-context=third.soroko-gcp.altoros.com`

1. Turn down the federation control plane:
    * `kubectl delete ns federation-system --context=first.soroko-gcp.altoros.com`

1. Destroy all clusters:
    * `kops delete cluster first.soroko-gcp.altoros.com --yes`
    * `kops delete cluster second.soroko-gcp.altoros.com --yes`
    * `kops delete cluster third.soroko-gcp.altoros.com --yes`
