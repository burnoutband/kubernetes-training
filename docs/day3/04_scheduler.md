## Scheduler

### Exercise 1:  Disabling the scheduler

1. SSH to the master node.

1. Move `kube-scheduler.manifest`  out of the `/etc/kubernetes/manifests/`

1. Wait until the kubelet shuts down the scheduler pod.

    Use the following command to list all system pods
    ```
    kubectl --namespace kube-system get pods
    ```

1. Deploy some pod normally.

1. Use `get pods` command to list all pods. The one that you've just deployed should be in a Pending state with no node assigned to it.

1. Use `describe pod` command to check which node is assigned to the pod.

1. Return the schedules definition back to the `/etc/kubernetes/manifests/` folder on the master node.

1. Wait until the scheduler runs and make sure that now a node is assigned to the pod and the pod is running.


### Exercise 2: Manually schedule a pod 

1. While the default schedulrer is disabled, try to manually assign a node to the container using API. 
    * Use curl to sent a POST requiest to the `{namespace}/pods/{name}/binding` enpoint. 
    * The body of the request should have the following format `{"apiVersion":"v1", "kind": "Binding", "metadata": {"name": "<pod-name>"}, "target": {"apiVersion": "v1", "kind": "Node", "name": "<node-name>"}}`
    * Use [this](https://kubernetes-v1-4.github.io/docs/api-reference/v1/operations/) and [this](https://kubernetes.io/blog/2017/03/advanced-scheduling-in-kubernetes/) docs for referennce.

