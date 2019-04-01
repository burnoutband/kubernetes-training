# Downward API

## Exercise 01: Expose namespace as environmental variable

Namespace name is an attribute of the pod. To expose it as environmental variable `MY_POD_NAMESPACE`:

1. Create file dapi-pod.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: dapi-envars-fieldref
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: [ "sh", "-c"]
      args:
      - while true; do
          echo -en '\n';
          printenv MY_POD_NAMESPACE;
          sleep 10;
        done;
      env:
        - name: MY_POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
  restartPolicy: Never
```

2. Create pod

```
kubectl apply -f dapi-pod.yaml
```

3. Verify the container in the pod is running

```
kubectl get pods
```

4. View the container's logs

```
kubectl logs dapi-envars-fieldref
```

The namespace name will be printed in the output.

5. Next, let's get inside the container

```
kubectl exec -it dapi-envars-fieldref -- sh
```
6. View the environment variables from the shell

```
/# printenv
```

## Exercise 02: Expose namespace as a file

Namespace name is an attribute of the pod. To expose it as file `/etc/podinfo/namespace`:

1. Create file dapi-volume.yaml

```
apiVersion: v1
kind: Pod
metadata:
name: kubernetes-downwardapi-volume-example
spec:
containers:
  - name: client-container
    image: k8s.gcr.io/busybox
    command: ["sh", "-c"]
    args:
    - while true; do
        if [[ -e /etc/podinfo/labels ]]; then
          echo -en '\n\n'; cat /etc/podinfo/namespace; fi;
        sleep 5;
      done;
    volumeMounts:
      - name: podinfo
        mountPath: /etc/podinfo
        readOnly: false
volumes:
  - name: podinfo
    downwardAPI:
      items:
        - path: "namespace"
          fieldRef:
            fieldPath: metadata.namespace
```

2. Create pod

```
kubectl apply -f dapi-volume.yaml
```

3. Verify the container in the pod is running

```
kubectl get pods
```

4. View the container's logs

```
kubectl logs kubernetes-downwardapi-volume-example
```

The namespace name will be printed in the output.

5. Next, let's get inside the container

```
kubectl exec -it kubernetes-downwardapi-volume-example -- sh
```
6. View the environment variables from the shell

```
/# cat /etc/podinfo/namespace
```

## Exercise 03: Expose container limits

Container limits are not properties of the pod but container running inside this pod. The syntax will be slightly different:

1. Create file dapi-envvars-container.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: dapi-envars-resourcefieldref
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox:1.24
      command: [ "sh", "-c"]
      args:
      - while true; do
          echo -en '\n';
          printenv MY_CPU_REQUEST MY_CPU_LIMIT;
          printenv MY_MEM_REQUEST MY_MEM_LIMIT;
          sleep 10;
        done;
      resources:
        requests:
          memory: "32Mi"
          cpu: "125m"
        limits:
          memory: "64Mi"
          cpu: "250m"
      env:
        - name: MY_CPU_REQUEST
          valueFrom:
            resourceFieldRef:
              containerName: test-container
              resource: requests.cpu
        - name: MY_CPU_LIMIT
          valueFrom:
            resourceFieldRef:
              containerName: test-container
              resource: limits.cpu
        - name: MY_MEM_REQUEST
          valueFrom:
            resourceFieldRef:
              containerName: test-container
              resource: requests.memory
        - name: MY_MEM_LIMIT
          valueFrom:
            resourceFieldRef:
              containerName: test-container
              resource: limits.memory
  restartPolicy: Never
```

2. Create pod

```
kubectl apply -f dapi-envars-container.yaml
```

3. Verify the container in the pod is running

```
kubectl get pods
```

4. View the container's logs

```
kubectl logs dapi-envars-resourcefieldref
```

Container limits and requests will be printed in the output both for CPU and memory.

## Exercise 04: Expose the pods name and owner (optional)

1. Attach label named `owner` to the pod
2. Print the pod's name and owner label value to the log output

You can use the same busybox image as in previous examples.

You can expose data either using enviornmental variables or files.

Hint: `metadata.name` field contains the pod's name
