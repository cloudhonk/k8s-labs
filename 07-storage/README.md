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

### Types of Persistent Volume
- local - Devices mounted locally to the cluster
- hostPath - Stores data within a named directory on a Node
- nfs - Used to access NFS mounts
- iscsi - iSCSI (SCSI over IP)
- csi - Integration with storage provider that supports the Container Storage Interface
- cephfs - Allow the use of CephFS Volumes
- fc - Fiber Channel (FC)

### The Lifecycle of PVs and PVCs
- Provisioning
- Binding
- Using
- Reclaiming

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
