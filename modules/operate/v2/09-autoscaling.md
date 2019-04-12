# Autoscaling 

## Module Objectives

- Use horizontal pod autoscaler

---

## Use Horizontal Autoscaler

Now we will use the horizontal autoscaler to automatically set the number of backend instances based on the current load.

1. Scale the number of backend instances to 1.

    > Note: You can either modify `manifests/backend.yaml` file and apply changes or use `kubectl scale` command.

1. Apply autoscaling to the backend Deployment.

    ```shell
    kubectl autoscale deployment backend --cpu-percent=50 --min=1 --max=3
    ```

1. Check the status of the autoscaler.

    ```shell
    kubectl get hpa
    ```

    ```
    NAME      REFERENCE            TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
    backend   Deployment/backend   0%/50%    1         3         1          1m
    ```

1. Exec inside the Pod.

    ```shell
    kubectl exec -it <backend-pod-name> bash
    ```

1. Install `stress` and use the following command to generate some load.

    ```shell
    apt-get update & apt-get install stress
    stress --cpu 60 --timeout 200
    ```

1. In a different terminal window watch the autoscaler status.

    ```shell
    watch kubectl get hpa
    ```

    ```
    NAME      REFERENCE            TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
    backend   Deployment/backend   149%/50%   1         3         3          13m
    ```

    Wait until the autoscaler scales the number of `backend` Pods to 3.

1. Save the autoscaler definition as a Kubernetes object and examine its content.

    ```shell
    kubectl get hpa -o yaml > manifests/autoscaler.yaml
    ```


