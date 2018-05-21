## etcd

etcd is a distributed key-value store that provides a reliable way to store data across a cluster of machines. It’s open-source and available on GitHub. etcd gracefully handles leader elections during network partitions and will tolerate machine failure, including the leader.

Kubernetes uses etcd to store all persistent information that it needs to operate (cluster state, current and desired state for all pods and deployments, secrets, config maps ...)

### Exercise 1: Manually access etcd 

etcd, as most of the kubernetes system components, runs inside a static pod. This means that we can use kubectl to access it.

1. Run the following command to list all system pods.
    ```
    kubectl --namespace kube-system get pods
    ```
    As you might see, there are 2 etcd pods in the list: `etcd-master` and `etcd-events`. The database itself is hosted inside `etcd-master` pod.

1. Exec inside etcd pod.
    ```
    kubectl --namespace kube-system exec -it etcd-server-master-us-west1-b-z2sr sh
    ```

1. Now you can use `etcdctl` to access etcd database. `etcdctl ls` command can be used to navigate inside etcd.
    ```
    etcdctl ls 
    etcdctl ls /registry
    ```
1. List all pods in default namespace
    ```
    etcdctl ls /registry/pods/default
    ```

1. Select some pod and get its spect from etcd database
    ```
    etcdctl get /registry/pods/default/<pod-name>
    ```

### Exercise 2: Backup etcd 

1. Follow the instructions in the etcd documentation to create etcd backup [Reference link](https://coreos.com/etcd/docs/latest/v2/admin_guide.html#disaster-recovery) 

