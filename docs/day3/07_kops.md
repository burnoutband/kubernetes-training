## kops deep dive


### Exercise 1: Installing Kubernetes Addons 

1. Install the [dashboard](https://github.com/kubernetes/dashboard) addon 
    ```
    kubectl create -f https://raw.githubusercontent.com/kubernetes/kops/master/addons/kubernetes-dashboard/v1.8.3.yaml
    ```

1. Navigate to https://<cluster-load-balancer-ip>/ui

1. The login credentials are:
    * Username: `admin`
    * Password: get by running `kops get secrets kube --type secret -oplaintext` or `kubectl config view --minify`


### Exercise 2: Switching cluster network provider 

1. Run the following command to edit cluster configuration
    ```
    kops edit cluster
    ```

1. Find `networking` section

1. Edit it to switch network provider
    ```
      networking:
        calico: {}
    ```

1. Apply the changes
    ```
    $ kops update cluster # to preview
    $ kops update cluster --yes # to apply
    $ kops rolling-update cluster # to preview the rolling-update
    $ kops rolling-update cluster --yes # to roll all your instances
    ```
    Pay attention to how kops drains all pods from the node that is being updated. This allows it no make the update without app downtime.

1. SSH to the master node and make sure that now kubelet runs with `--network-plugin=cni` argument.

1. Deploy 2 pods that talks to each other, to make sure that network provider is working correctly.

### Exercise 3 (Optional): Deploy HA cluster 

1. Deploy a new cluster. Follow the instructions in the [kops documentation](https://github.com/kubernetes/kops/blob/master/docs/high_availability.md)
1. Delete the second cluster.

