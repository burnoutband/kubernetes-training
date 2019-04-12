# Health Checks

## Module Objectives

- Define custom health checks and liveness probes

---
## Define Custom Health Checks and Liveness Probes

By default, Kubernetes assumes that a Pod is ready to accept requests as soon as its container is ready and the main process starts running. Once this condition is met, Kubernetes Services will redirect traffic to the Pod. This can cause problems if the application needs some time to initialize. Let's reproduce this behavior.

1. Add `-delay=60` to the `command` property in the `manifests/backend.yaml` file and apply changes. This will cause our app to sleep for a minute on startup.

1. Open the app in the web browser. You should see the following error for about a minute:

    ```
    Error: Get http://backend:8080: dial tcp 10.51.249.99:8080: connect: connection refused
    ```

Now let's fix this problem by introducing a `readinessProbe` and a `livenessProbe`.

The `readinessProbe` and `livenessProbe` are used by Kubernetes to check the health of the containers in a Pod. The probes can check either HTTP endpoints or run shell commands to determine the health of a container. The difference between readiness and liveness probes is subtle, but important.

If an app fails a `livenessProbe`, kubernetes will restart the container.

> Caution: A misconfigured `livenessProbe` could cause deadlock for an application to start. If an application takes more time than the probe allows, then the livenessProbe will always fail and the app will never successfully start. See [this document](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) for how to adjust the timing on probes.

If an app fails a `readinessProbe`, Kubernetes will consider the container unhealthy and send traffic to other Pods.

> Note: Unlike liveness, the Pod is not restarted.

For this exercise, we will use only a `readinessProbe`.

1. Edit `manifests/backend.yaml`, add the following section into it and apply the changes.

    ```yaml
            readinessProbe:
              httpGet:
                path: /healthz
                port: 8080
    ```

    This section should go under `spec -> template -> spec -> containers[name=backend]` and should be aligned with `image`, `env` and `command` properties.

1. Run `watch kubectl get pod` to see how Kubernetes is rolling out your Pods. This time it will do it much slower, making sure that previous Pods are ready before starting a new set of Pods.
