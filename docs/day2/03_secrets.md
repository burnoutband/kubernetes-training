## Secrets

Objects of type secret are intended to hold sensitive information, such as passwords, OAuth tokens, and ssh keys. Putting this information in a secret is safer and more flexible than putting it verbatim in a pod definition or in a docker image

### Exercise 1: Storing secrets in k8s

1. Create file with secrets in Cloud Shell 
    ```
    $ echo -n 'Ood7ooch8a' > ./password.txt
    ```

1. Create secret in k8s from file
    ```
    $ kubectl create secret generic password --from-file=./password.txt
    secret "password" created
    ```

1. Get information about created secrets in k8s
    ```
    $ kubectl get secrets
    NAME                   TYPE                                  DATA      AGE
    password               Opaque                                1         12s
    ```

1. Describe previously created secret
    ```
    $ kubectl describe secrets/password
    Name:         password
    Namespace:    default
    Labels:       <none>
    Annotations:  <none>

    Type:  Opaque

    Data
    ====
    password.txt:  10 bytes
    ```

1. Create pod with attached secret password file
    ```
    $ kubectl create -f pod.yml
    ```
    Pod config file is listed below.
    ```
    $ cat pod.yml
    apiVersion: v1
    kind: Pod
    metadata:
      name: secretpod
    spec:
      containers:
      - name: secretpod
        image: redis
        volumeMounts:
        - name: pass
          mountPath: "/tmp/pass"
          readOnly: true
      volumes:
      - name: pass
        secret:
          secretName: password
    ```

1. Login to created Pod
    ```
    $ kubectl exec -it secretpod -- /bin/bash
    ```

1. Get password
    ```
    # cat /tmp/pass/password.txt
    Ood7ooch8a
    ```
