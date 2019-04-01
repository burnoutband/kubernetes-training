# Sidecars and Init Containers

## Module Objectives

1. Use pod/node affinity and anti-affinity

---
## Use Pod/Node Affinity and Anti-affinity

By default the Kubernetes scheduler will try to evenly distribute pods between nodes, taking into consideration current node resource utilization as well as other criteria. But there are situations when you want to influence how Pods are placed. For example, let's imagine that we want all our 3 Pods to be colocated on a single node (this could make sense if we care more about performance than high availability). We don't want to pick a particular node, instead we prefer the scheduler to do this for us. We also don't want this rule to be very strict, if there is not such a node that has enough resources to host all of our Pods we want to allow the scheduler to break this rule and put Pods on different nodes. This is the kind of rule that can be expressed using [node/pod affinity](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity).

1. Run the following command to make sure that your Pods are allocated on different worker nodes (most likely this will be the case).

    ```shell
    kubectl get pod -o wide
    ```

    ```
    NAME       READY     STATUS    RESTARTS   AGE       IP           NODE
    backend    1/1       Running   0          10m       10.48.1.28   gke-gke-workshop-0-default-pool-c70d26ac-gvpw
    db         1/1       Running   0          15m       10.48.1.26   gke-gke-workshop-0-default-pool-c70d26ac-gvpw
    frontend   1/1       Running   0          20h       10.48.0.11   gke-gke-workshop-0-default-pool-c70d26ac-p2x2
    ```

1. Add the following block to the `spec` element for all 3 of our Pods (this element should be aligned with the same number of spaces as `containers` key).

    ```yaml
    affinity:
      podAffinity:
        preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 100
          podAffinityTerm:
            labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - gceme
            topologyKey: "kubernetes.io/hostname"
    ```

    Refer to the [official documentation](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#inter-pod-affinity-and-anti-affinity-beta-feature) to make sure you understand the meaning of all specified properties.

1. Delete all Pods.

    ```shell
    kubectl delete pod --all
    ```

1. Recreate all Pods.

    ```shell
    kubectl apply -R -f manifests/
    ```

1. Make sure all Pods are colocated on a single node now.

    ```shell
    kubectl get pod -o wide
    ```

    ```
    NAME       READY     STATUS    RESTARTS   AGE       IP           NODE
    backend    1/1       Running   0          5m        10.48.1.30   gke-gke-workshop-0-default-pool-c70d26ac-gvpw
    db         1/1       Running   0          5m        10.48.1.29   gke-gke-workshop-0-default-pool-c70d26ac-gvpw
    frontend   1/1       Running   0          5m        10.48.1.31   gke-gke-workshop-0-default-pool-c70d26ac-gvpw
    ```

