## Storing secrets in k8s

Create file with secrets on local machine
```
$ echo -n 'Ood7ooch8a' > ./password.txt
```
Create secret in k8s from file
```
$ kubectl create secret generic password --from-file=./password.txt
secret "password" created
```
Get information about created secrets in k8s
```
$ kubectl get secrets
NAME                   TYPE                                  DATA      AGE
password               Opaque                                1         12s
```
Describe previously created secret
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
Create pod with attached secret password file
```
$ kubectl create -f pod.yml
```
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
Login to created Pod
```
$ kubectl exec -it secretpod -- /bin/bash
```
Get password
```
# cat /tmp/pass/password.txt
Ood7ooch8a
```
