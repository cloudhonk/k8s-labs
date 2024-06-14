## K8s Volume

**NOTES**
- A Pod can use any number of volume types simultaneously. 
- Ephemeral volume types have a lifetime of a pod, but persistent volumes exist beyond the lifetime of a pod. - When a pod ceases to exist, Kubernetes destroys ephemeral volumes; however, Kubernetes does not destroy persistent volumes. 
- For any kind of volume in a given pod, data is preserved across container restarts.
- At its core, a volume is a directory, possibly with some data in it, which is accessible to the containers in a pod. How that directory comes to be, the medium that backs it, and the contents of it are determined by the particular volume type used.
- Volumes cannot mount within other volumes (but see Using [subPath](https://kubernetes.io/docs/concepts/storage/volumes/#using-subpath) for a related mechanism). 
- Also, a volume cannot contain a hard link to anything in a different volume.

## Types of Volume

- configMap

Deploy ConfigMap volumes
```shell
# deploy
kubectl apply -f configs/configmap-volume.yaml

# verify
kubectl exec configmap-pod -- cat /etc/config/log_level

## Cleanup
kubectl delete -f configs/configmap-volume.yaml 
```

- Secrets

Deploy Secret volumes
```shell
# deploy
kubectl apply -f configs/secret-volume.yaml

# verify
kubectl exec secret-pod -- cat /etc/top/secret

## Cleanup
kubectl delete -f configs/secret-volume.yaml 
```
- emptyDir

**NOTES**

  - `emptyDir volume` is ephemeral volume, meaning when pod is removed from a nod, the data in the emptyDir is deleted permanently.
  - The data in an emptyDir volume is safe across container crashes.
  - `emptyDir volume` is created when the Pod is assigned to a node. 
  - The emptyDir volume is initially empty. 
  - All containers in the Pod can read and write the same files in the emptyDir volume.

```shell
# deploy the pod
kubectl apply -f configs/emptyDir-volume.yaml

# verify
kubectl exec emptydir-pod -- ls -la /cache

# delete the pod
kubectl delete -f configs/emptyDir-volume.yaml
```

- hostPath

**NOTES**

- A hostPath volume mounts a file or directory from the host node's filesystem into your Pod. This is not something that most Pods will need, but it offers a powerful escape hatch for some applications.

- Using the hostPath volume type presents many security risks. If you can avoid using a hostPath volume, you should. For example, define a local PersistentVolume, and use that instead.

**Usecases**:

- running a container that needs access to node-level system components (such as a container that transfers system logs to a central location, accessing those logs using a read-only mount of /var/log)

- providing configuration stored in the host file system for static pods. Unlike normal Pods, static Pods cannot access ConfigMaps


  SubType
  - "" 
  - DirectoryOrCreate
  - Directory
  - FileOrCreate
  - File
  - Socket
  - CharDevice
  - BlockDevice

```shell
# deploy
kubectl apply -f configs/hostPath-volume.yaml

# verify
kubectl exec -it hostpath-pod -- ls -la -R /var/local/

# delete
kubectl delete -f configs/hostPath-volume.yaml 
```
- downwardAPI

Allows containers to access runtime data about themselves.

```shell
# deploy
kubectl apply -f configs/downwardapi-volume.yaml

# verify
kubectl exec downwardapi-pod -- cat /etc/podinfo/labels
kubectl exec downwardapi-pod -- cat /etc/podinfo/annotations

# delete
kubectl delete -f configs/downwardapi-volume.yaml
```
- projected

A projected volume maps several existing volume sources into the same directory. Currently, the following types of volume sources can be projected:

  - secret
  - downwardAPI
  - configMap
  - serviceAccountToken
  - clusterTrustBundle

```shell
# deploy
kubectl apply -f configs/projected-volume.yaml

# verify
kubectl exec projected-pod -- ls -la /projected-volume
kubectl exec projected-pod -- cat /projected-volume/my-group/my-username
kubectl exec projected-pod -- cat /projected-volume/my-group/my-config
kubectl exec projected-pod -- cat /projected-volume/labels
kubectl exec projected-pod -- cat /projected-volume/cpu_limit

# delete
kubectl delete -f configs/projected-volume.yaml
```

- persistentVolumeClaim
- local
- nfs
- iscsi
- fc (fibre channel)
- gcePersistentDisk (deprecated) 
- gitRepo (deprecated)
- glusterfs (removed)
- portworxVolume (deprecated)
- Portworx CSI migration
- rbd
- RBD CSI migration
- awsElasticBlockStore (deprecated)
- azureDisk (deprecated)
- azureFile (deprecated)
- azureFile CSI migration
- azureFile CSI migration complete
- cephfs (deprecated)
- cinder (deprecated)
- vsphereVolume (deprecated) 
- vSphere CSI migration
- vSphere CSI migration complete

## Kubernetes Persistent Volumes

Kubernetes Persistent Volumes(PVs) provide storage for the application's Pods. Data are written to a
volume is managed independently of the Pods that access it, ensuring the data remains available after Pod restarts and failures.

PVs work in conjunction with Persistent Volume Claims (PVCs), another type of object that permits Pods to request access to PVs. To successfully utilize persistent storage in a cluster, you'll need a PV and a PVC that connects it to your Pod.

- A PersistentVolume (PV) is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes.
- A PersistentVolumeClaim (PVC) is a request for storage by a user.

### Lifecycle of Persistent Volume

- Provisioning: 
  
  dynamic/static

- Binding: 

A control loop in the control plane watches for new PVCs, finds a matching PV (if possible), and binds them together. If a PV was dynamically provisioned for a new PVC, the loop will always bind that PV to the PVC.

Claims will remain unbound indefinitely if a matching volume does not exist.

A PVC to PV binding is a one-to-one mapping

- Using

Pods use claims as volumes.

For volumes that support multiple access modes, the user specifies which mode is desired when using their claim as a volume in a Pod.

Once a user has a claim and that claim is bound, the bound PV belongs to the user for as long as they need it.

- Storage Object in Use Protection

Ensure PersistentVolumeClaims (PVCs) in active use by a Pod and PersistentVolume (PVs) that are bound to PVCs are not removed from the system, as this may result in data loss.

If a user deletes a PVC in active use by a Pod, the PVC is not removed immediately. PVC removal is postponed until the PVC is no longer actively used by any Pods. 

Also, if an admin deletes a PV that is bound to a PVC, the PV is not removed immediately. PV removal is postponed until the PV is no longer bound to a PVC.

- Reclaiming

The reclaim policy for a PersistentVolume tells the cluster what to do with the volume `after it has been released of its claim`. Currently, volumes can either be `Retained`, `Recycled`, or `Deleted`.

Retain: The Retain reclaim policy allows for manual reclamation of the resource. When the PersistentVolumeClaim is deleted, the PersistentVolume still exists and the volume is considered "released". But it is not yet available for another claim because the previous claimant's data remains on the volume.

Delete: For volume plugins that support the Delete reclaim policy, deletion removes both the PersistentVolume object from Kubernetes, as well as the associated storage asset in the external infrastructure.

Recycle: The Recycle reclaim policy is deprecated. Instead, the recommended approach is to use dynamic provisioning. If supported by the underlying volume plugin, the Recycle reclaim policy performs a basic scrub (`rm -rf /thevolume/*`) on the volume and makes it available again for a new claim. 

- Reserving a PersistentVolume

By specifying a PersistentVolume in a PersistentVolumeClaim, you declare a binding between that specific PV and PVC. If the PersistentVolume exists and has not reserved PersistentVolumeClaims through its claimRef field, then the PersistentVolume and PersistentVolumeClaim will be bound.

- Expanding PVCs

You can only expand a PVC if its storage class's allowVolumeExpansion field is set to true.


### Types of Persistent Volume
- local - Devices mounted locally to the cluster
- hostPath - (for single node testing only; WILL NOT WORK in a multi-node cluster; consider using local volume instead)
- nfs - Used to access NFS mounts
- iscsi - iSCSI (SCSI over IP)
- csi - Integration with storage provider that supports the Container Storage Interface
- cephfs - Allow the use of CephFS Volumes
- fc - Fiber Channel (FC)

### Presitent Volume configurations

**Capacity:**

  ```yaml
  spec:
    capacity:
      storage: 5Gi
  ```

**Volume Mode:**

Filesystem is the default mode used when volumeMode parameter is omitted.

A volume with volumeMode: Filesystem is mounted into Pods into a directory. If the volume is backed by a block device and the device is empty, Kubernetes creates a filesystem on the device before mounting it for the first time.

```yaml
spec:
  volumeMode:
    storage: FileSystem #Block
```

**Access Modes(Required):**

A PersistentVolume can be mounted on a host in any way supported by the resource provider. Kubernetes uses volume access modes to match PersistentVolumeClaims and PersistentVolumes. 

```yaml
spec:
  accessModes:
    - ReadWriteOnce #ReadOnlyMany, ReadWriteMany, ReadWriteOncePod
```

- `ReadWriteOnce`
the volume can be mounted as read-write by a single node. ReadWriteOnce access mode still can allow multiple pods to access the volume when the pods are running on the same node. For single pod access, please see `ReadWriteOncePod`.

- `ReadOnlyMany`
the volume can be mounted as read-only by many nodes.

- `ReadWriteMany`
the volume can be mounted as read-write by many nodes.

- `ReadWriteOncePod`
the volume can be mounted as read-write by a single Pod. Use ReadWriteOncePod access mode if you want to ensure that only one pod across the whole cluster can read that PVC or write to it.

**NOTES**

- Volume access modes do not enforce write protection once the storage has been mounted. Even if the access modes are specified as ReadWriteOnce, ReadOnlyMany, or ReadWriteMany, they don't set any constraints on the volume.
- A volume can only be mounted using one access mode at a time, even if it supports many.

**Class:**

A PV can have a class, which is specified by setting the storageClassName attribute to the name of a StorageClass. A PV of a particular class can only be bound to PVCs requesting that class. A PV with no storageClassName has no class and can only be bound to PVCs that request no particular class.

In the past, the annotation `volume.beta.kubernetes.io/storage-class` was used instead of the storageClassName attribute. This annotation is still working; however, it will become fully deprecated in a future Kubernetes release.


**Reclaim Policy:**

- Retain -- manual reclamation
- Recycle -- basic scrub (rm -rf /thevolume/*)
- Delete -- delete the volume

**Mount Options:**

A Kubernetes administrator can specify additional mount options for when a Persistent Volume is mounted on a node. Mount options are not validated. If a mount option is invalid, the mount fails.

The following volume types support mount options. The list doesn't include deprecated types:

- azureFile
- iscsi
- nfs
- vsphereVolume

**Node Affinity:**

A PV can specify node affinity to define constraints that limit what nodes this volume can be accessed from. Pods that use a PV will only be scheduled to nodes that are selected by the node affinity. To specify node affinity, set nodeAffinity in the .spec of a PV.

NOTE: For most volume types, you do not need to set this field. You need to explicitly set this for local volumes.

**Phase:**

Available, Bound, Released, Failed

### PersistentVolumeClaims configurations

**Access Modes:**

Claims use the same conventions as volumes when requesting storage with specific access modes.

**Volume Modes:**

Claims use the same convention as volumes to indicate the consumption of the volume as either a filesystem or block device.

**Resources:**

Claims, like Pods, can request specific quantities of a resource. In this case, the request is for storage. The same resource model applies to both volumes and claims.

```yaml
spec:
  resources:
    requests:
      storage: 8Gi
```


**Selector:**

Claims can specify a label selector to further filter the set of volumes. Only the volumes whose labels match the selector can be bound to the claim. The selector can consist of two fields:

- matchLabels
- matchExpressions

```yaml
spec:
  selector:
    matchLabels:
      release: "stable"
    matchExpressions:
      - {key: environment, operator: In, values: [dev]}
```

**Class:**

A claim can request a particular class by specifying the name of a StorageClass using the attribute storageClassName. Only PVs of the requested class, ones with the same storageClassName as the PVC, can be bound to the PVC.

PVCs don't necessarily have to request a class. A PVC with its storageClassName set equal to "" is always interpreted to be requesting a PV with no class, so it can only be bound to PVs with no class (no annotation or one set equal to ""). A PVC with no storageClassName is not quite the same and is treated differently by the cluster.

```yaml
spec:
  storageClassName: slow
```


### Create a Persistent Volume
```bash
kubectl apply -f configs/pv.yaml
```

### Create a Persistent Volume Claim
```bash
kubectl apply -f configs/pvc.yaml
```

### Get PV Details
```bash
kubectl get pv
```

### Get PVC Details
```bash
kubectl get pvc
```

## Storage Classes
A StorageClass provides a way for administrators to describe the classes of storage they offer.

Different classes might map to quality-of-service levels, or to backup policies, or to arbitrary policies determined by the cluster administrators.

When a PVC does not specify a storageClassName, the default StorageClass is used.

### StorageClass objects

Each StorageClass contains the fields `provisioner`, `parameters`, and `reclaimPolicy`, which are used when a PersistentVolume belonging to the class needs to be dynamically provisioned to satisfy a PersistentVolumeClaim (PVC).

Sample

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: low-latency
  annotations:
    storageclass.kubernetes.io/is-default-class: "false"
provisioner: csi-driver.example-vendor.example
reclaimPolicy: Retain # default value is Delete
allowVolumeExpansion: true
mountOptions:
  - discard # this might enable UNMAP / TRIM at the block storage layer
volumeBindingMode: WaitForFirstConsumer
parameters:
  guaranteedReadWriteLatency: "true" # provider-specific
```

**Default StorageClass:**

If you set the `storageclass.kubernetes.io/is-default-class` annotation to true on more than one StorageClass in your cluster, and you then create a PersistentVolumeClaim with no storageClassName set, Kubernetes uses the most recently created default StorageClass.

**Provisioner:**

Each StorageClass has a provisioner that determines what volume plugin is used for provisioning PVs. This field must be specified.

**Reclaim policy:**

PersistentVolumes that are dynamically created by a StorageClass will have the reclaim policy specified in the reclaimPolicy field of the class, which can be either Delete or Retain.

**Volume expansion:**

PersistentVolumes can be configured to be expandable. This allows you to resize the volume by editing the corresponding PVC object, requesting a new larger amount of storage.

**NOTE:** You can only use the volume expansion feature to grow a Volume, not to shrink it.

**Mount options:**

PersistentVolumes that are dynamically created by a StorageClass will have the mount options specified in the mountOptions field of the class.

**Volume binding mode:**

The volumeBindingMode field controls when volume binding and dynamic provisioning should occur. When unset, `Immediate` mode is used by default.

The `Immediate` mode indicates that volume binding and dynamic provisioning occurs once the PersistentVolumeClaim is created.

`WaitForFirstConsumer` mode delays the binding and provisioning of a PersistentVolume until a Pod using the PersistentVolumeClaim is created.


**NOTE:** 

If you choose to use `WaitForFirstConsumer`, do not use `nodeName` in the `Pod spec` to specify node affinity. If `nodeName` is used in this case, the scheduler will be bypassed and PVC will remain in pending state.

Instead, you can use node selector for `kubernetes.io/hostname:`

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  nodeSelector:
    kubernetes.io/hostname: kube-01
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage
```

**Parameters:**

StorageClasses have parameters that describe volumes belonging to the storage class. Different parameters may be accepted depending on the provisioner. When a parameter is omitted, some default is used.

### Dynamic Provisioning of PVs
Static provisioning of PVs, as shown above, is cumbersome: you must create the PV, and then the PVC while ensuring their properties match.

Dynamic provisioning is a more popular option for real-world use cases. The PV is created dynamically based on the PVC's configuration.

### Create a Storage Class with NFS Driver
```bash
kubectl apply -f configs/storageclass.yaml
```

### Get the configured storage class information
```bash
kubectl get sc
```

### Create a Dynamic PV
```bash
kubectl apply -f configs/pvc-dynamic.yaml
```

### Get the details of PV
```bash
kubectl get pv
```

### Attach PVCs to the Pods
Once PV and PVC are configured, now it is time to attach PVC to a Pod. PVC will mount the PV into the Pod's filesystem, allowing the Pod to read and write files with full persistence.

```bash
kubectl apply -f configs/pod.yaml
```

### Now we will check the persistence of the data:

Create the Pod, then we will write something to the /data

```
# Get a shell inside the Pod
kubectl exec -it pod/k8s-labs -- bash

# No files in the volume yet
root@k8s-labs:/# ls /data

# Write a file to the volume
root@k8s-labs:/# echo "bar" > /data/foo

# The file now shows in the volume
root@k8s-labs:/# ls /data
```

The files written to the volume are now stored independently of the Pod. We can delete the Pod and recreate it, data will still be intact within the PV:

```
kubectl delete pods/pvc-pod

kubectl apply -f configs/pod.yaml

kubectl exec -it pod/k8s-labs -- bash

root@k8s-labs:/# cat /data/foo
# We should see `bar` as a output.
```

## Remove the Labs Resources
```bash
kubectl delete -f configs/
```
