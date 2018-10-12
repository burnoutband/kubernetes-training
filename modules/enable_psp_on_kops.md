## Enable Pod Security Policies on KOPS

### Edit cluster config

```
kops edit cluster
```
Add to the bottom of the `spec` section and save.
```
  kubeAPIServer:
    admissionControl:
    - NamespaceLifecycle
    - LimitRanger
    - ServiceAccount
    - PersistentVolumeLabel
    - DefaultStorageClass
    - ResourceQuota
    - PodSecurityPolicy
    - DefaultTolerationSeconds
```

### Run update cluster

Running below should show you changes that will be added to cluster.
```
kops update cluster
```

```
Will create resources:
  InstanceTemplate/master-us-west1-c-simple-k8s-local
        NamePrefix              master-us-west1-c-simple-k8s-local
        Network                 name:default id:default
        Tags                    [simple-k8s-local-k8s-io-role-master]
        Preemptible             false
        BootDiskImage           cos-cloud/cos-stable-60-9592-90-0
        BootDiskSizeGB          64
        BootDiskType            pd-standard
        CanIPForward            true
        Scopes                  [compute-rw, monitoring, logging-write, storage-ro, https://www.googleapis.com/auth/ndev.clouddns.readwrite]
        Metadata                {startup-script: <resource>, cluster-name: <resource>, ssh-keys: <resource>}
        MachineType             n1-standard-1

  ManagedFile/simple.k8s.local-addons-podsecuritypolicy.addons.k8s.io-k8s-1.10
        Location                addons/podsecuritypolicy.addons.k8s.io/k8s-1.10.yaml

  ManagedFile/simple.k8s.local-addons-podsecuritypolicy.addons.k8s.io-k8s-1.9
        Location                addons/podsecuritypolicy.addons.k8s.io/k8s-1.9.yaml

Will modify resources:
  InstanceGroupManager/c-master-us-west1-c-simple-k8s-local
        InstanceTemplate         id:master-us-west1-c-simple-k8s-local-1539365360 -> name:master-us-west1-c-simple-k8s-local

  ManagedFile/simple.k8s.local-addons-bootstrap
        Contents
                                ...
                                        k8s-addon: core.addons.k8s.io
                                      version: 1.4.0
                                +   - id: k8s-1.9
                                +     kubernetesVersion: '>=1.9.0 <1.10.0'
                                +     manifest: podsecuritypolicy.addons.k8s.io/k8s-1.9.yaml
                                +     name: podsecuritypolicy.addons.k8s.io
                                +     selector:
                                +       k8s-addon: podsecuritypolicy.addons.k8s.io
                                +     version: 0.0.4
                                +   - id: k8s-1.10
                                +     kubernetesVersion: '>=1.10.0'
                                +     manifest: podsecuritypolicy.addons.k8s.io/k8s-1.10.yaml
                                +     name: podsecuritypolicy.addons.k8s.io
                                +     selector:
                                +       k8s-addon: podsecuritypolicy.addons.k8s.io
                                +     version: 0.0.4
                                    - id: pre-k8s-1.6
                                      kubernetesVersion: <1.6.0
                                ...


Must specify --yes to apply changes
```

```
kops update cluster --yes
```

```
Using cluster from kubectl context: simple.k8s.local

I1012 11:32:01.301048     439 apply_cluster.go:505] Gossip DNS: skipping DNS validation
W1012 11:32:01.328527     439 external_access.go:36] TODO: Harmonize gcemodel ExternalAccessModelBuilder with awsmodel
W1012 11:32:01.328873     439 firewall.go:35] TODO: Harmonize gcemodel with awsmodel for firewall - GCE model is way too open
W1012 11:32:01.329051     439 firewall.go:63] Adding overlay network for X -> node rule - HACK
W1012 11:32:01.329186     439 firewall.go:64] We should probably use subnets?
W1012 11:32:01.329322     439 firewall.go:118] Adding overlay network for X -> master rule - HACK
W1012 11:32:01.569200     439 storageacl.go:71] we need to split master / node roles
I1012 11:32:04.570823     439 executor.go:103] Tasks: 0 done / 55 total; 28 can run
I1012 11:32:05.969269     439 executor.go:103] Tasks: 28 done / 55 total; 25 can run
I1012 11:32:06.141566     439 instancetemplate.go:214] We should be using NVME for GCE
I1012 11:32:06.144097     439 instancetemplate.go:214] We should be using NVME for GCE
I1012 11:32:06.170989     439 instancetemplate.go:214] We should be using NVME for GCE
I1012 11:32:11.078223     439 executor.go:103] Tasks: 53 done / 55 total; 2 can run
I1012 11:32:15.506781     439 executor.go:103] Tasks: 55 done / 55 total; 0 can run
I1012 11:32:15.835052     439 update_cluster.go:290] Exporting kubecfg for cluster
kops has set your kubectl context to simple.k8s.local

Cluster changes have been applied to the cloud.


Changes may require instances to restart: kops rolling-update cluster
```

We will need to run below to check for restart
```
kops rolling-update cluster
```

```
Using cluster from kubectl context: simple.k8s.local

I1012 11:32:47.710144     455 gce_cloud.go:273] Scanning zones: [us-west1-b us-west1-c us-west1-a]
NAME                    STATUS          NEEDUPDATE      READY   MIN     MAX     NODES
master-us-west1-c       NeedsUpdate     1               0       1       1       1
nodes                   Ready           0               2       2       2       2

Must specify --yes to rolling-update.
```

Once below is run, the cluster will be ready for enforcing pod security.
```
kops rolling-update cluster --yes
```
