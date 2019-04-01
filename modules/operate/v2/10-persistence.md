# Persistence

## Module Objectives

1. Use Persistent Volumes to store data
1. Convert a Persistent Volume to Persistent Volume Claim
1. Create a Storage Class
1. Use StatefulSet to deploy a mysql galera cluster

---

## Use a Persistent Volume to Store Data

1. Create a data disk.

    ```shell
    gcloud compute disks create \
      --size=50GB \
      --zone=us-west2-b \
      my-data-disk
    ```

    > Note: The disk must be in the same zone and project as GKE cluster.

1. Edit the database manifest to add the Volume to the Pod `spec`.

    ```yaml
    volumes:
    - name: my-data
      gcePersistentDisk:
        pdName: my-data-disk
        fsType: ext4
    ```

    > Note: Add it to the Pod spec not the Deployment spec!

1. Now mount the Volume inside the MySQL container.

    ```yaml
    containers:
      - image: mysql:5.6
        name: mysql
        volumeMounts:
        - mountPath: /var/lib/mysql
          name: my-data
    ```

1. Re-create the Deployment.

    ```shell
    kubectl apply -f manifests/db.yaml
    ```

1. Test the data persists between Pod restarts.

    * Add some notes to the frontend app and then delete the mysql Pod.

    * The Deployment Controller will automatically re-create the Pod and mount the Persistent Volume.

## Convert Persistent Volume to Persistent Volume Claim

A Persistent Volume Claim automates disk provisioning.

1. Create a Persistent Volume Claim for 50Gi disk `manifests/pvc.yaml`.

    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: dynamic-data
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 50Gi
    ```

1. Apply configuration.

    ```shell
    kubectl apply -f manifests/pvc.yaml
    ```

1. Verify PVC is created.

    ```shell
    kubectl describe pvc/dynamic-data
    ```

    ```
    Events:
      Type       Reason                 Age   From                         Message
      ----       ------                 ----  ----                         -------
      Normal     ProvisioningSucceeded  90s   persistentvolume-controller  Successfully provisioned volume pvc-5c3004bd-f476-11e8-aef3-42010a840052 using kubernetes.io/gce-pd
    ```

1. Let's look at the Volume.

    ```shell
    kubectl get pv
    ```

    ```
    NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS   REASON   AGE
    pvc-5c3004bd-f476-11e8-aef3-42010a840052   50Gi       RWO            Delete           Bound    default/dynamic-data   standard                2m
    ```

1. You may find the disk in the gCloud output as well.

    ```shell
    gcloud compute disks list
    ```

    ```
    gke-gke-workshop-73cb6-pvc-5c3004bd-f476-11e8-aef3-42010a840052  us-west2-b  50       pd-standard  READY
    ```

    Kubernetes provisioned 50Gi and made it available to use in Pods.

1. Reconfigure MySQL to use PVC instead of PV.

    The data will be lost - it is another disk!

    Edit the `manifests/db.yaml` file.

    ```yaml
    spec:
      volumes:
      - name: my-data
        persistentVolumeClaim:
          claimName: dynamic-data
      ```

1. You need to re-create the Pod or update the Deployment.

    ```shell
    kubectl delete -f manifests/db.yaml
    kubectl apply -f manifests/db.yaml
    ```

1. Find the line in the event stream from the Pod telling that Volume was mounted.

    ```shell
    kubectl describe pod db
    ```

    ```
    Normal  SuccessfulAttachVolume  107s  attachdetach-controller  AttachVolume.Attach succeeded for volume "pvc-5c3004bd-f476-11e8-aef3-42010a840052"
    ```

1. Go to frontend and verify the previous notes are gone. That mean you use a new disk to store data.

## Create a Storage Class

Google Cloud offers several disk types. They differ based on performance, reliability and price. How can I use particular disk type for Kubernetes workloads?

Kubernetes defines resource type called StorageClass. By specifying StorageClass in you PVCs you can provision different disk types.

1. Create a Storage Class that uses SSDs `manifests/storage-class.yaml`.

    ```yaml
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: ssd
    provisioner: kubernetes.io/gce-pd
    parameters:
      type: pd-ssd
    ```

    `type: pd-ssd` tells `gce-pd` provisioner to create an SSD disk instead of standard one.

1. Apply configuration.

    ```shell
    kubectl apply -f manifests/storage-class.yaml
    ```

1. Verify the StorageClass was created.

    ```shell
    kubectl get storageClass
    ```

    ```
    NAME                 PROVISIONER            AGE
    ssd                  kubernetes.io/gce-pd   5m
    standard (default)   kubernetes.io/gce-pd   1h
    ```

1. Now create PVC that uses this class `manifests/pvc-fast.yaml`.

    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: fast-data
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 50Gi
      storageClassName: ssd
    ```

    ```shell
    kubectl apply -f manifests/pvc-fast.yaml
    ```

    ```shell
    kubectl get pvc
    ```

    ```
    NAME           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    dynamic-data   Bound    pvc-5c3004bd-f476-11e8-aef3-42010a840052   50Gi       RWO            standard       38m
    fast-data      Bound    pvc-eaac23d7-f47a-11e8-aef3-42010a840052   50Gi       RWO            ssd            5m
    ```

1. The last step is to change the PVC reference from the database Pod and update the Deployment. Please do this by yourself.

