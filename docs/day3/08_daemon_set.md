# Daemon Set

In these exercises you will deploy fluentd logging agent to all the production nodes and optionally perform the rolling upgrade for them.

## Excercise 01: Deploy fluentd logging agent to all production nodes

DaemonSet allows to schedule pod to all the nodes satisfying the selector condition. This is useful when you implement logging agent, for instance. The pods scheduled with DaemonSet will start before pods scheduled with Deployment.

1. Create specification file `fluentd.yaml` for the fluentd agent

    ```
    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      name: fluentd
    spec:
      selector:
          matchLabels:
            name: fluentd # Label selector that determines which Pods belong to the DaemonSet
      template:
        metadata:
          labels:
            name: fluentd # Pod template's label selector
        spec:
          nodeSelector:
            type: prod # Node label selector that determines on which nodes Pod should be scheduled
                       # In this case, Pods are only scheduled to nodes bearing the label "type: prod"
          containers:
          - name: fluentd
            image: gcr.io/google-containers/fluentd-elasticsearch:1.20
            resources:
              limits:
                memory: 200Mi
              requests:
                cpu: 100m
                memory: 200Mi
    ```

1. Create the DaemonSet

    ```
    kubectl apply -f fluentd.yaml
    ```

1. List the available DaemonSets

    ```
    kubectl get daemonsets
    NAME      DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE   SELECTOR   AGE
    fluentd   0         0         0         0            0           type=prod     14s
    ```
You can see there are no scheduled pods. This is because we wanted them to run on nodes with the label `type=prod` only and there are no such nodes by default.

1. Let's label one node

```
kubectl get nodes
NAME      STATUS    ROLES     AGE       VERSION
master    Ready     master    24m       v1.10.0
node01    Ready     <none>    24m       v1.10.0

kubectl label node node01 type=prod
node "node01" labeled
```

1. Now DaemonSet will have a single pod scheduled to the `node01`

    ```
    kubectl get daemonsets
    NAME      DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
    fluentd   1         1         0         1            0           type=prod     1m
    ```

## Exercise 02 (optional): performing the rolling update

By default kubernetes scheduler uses `OnDelete` strategy for updating DaemonSets. That mean old pods will be running after template update until you manually delete them. In this exercise you will implement `RollingUpdate` strategy when pods are recreated in controlled fashion one by one.

Use official [documentation](https://kubernetes.io/docs/tasks/manage-daemon/update-daemon-set/) as your guide. Before starting please label other worker nodes with `type=prod` label so you can see them updated one by one.
