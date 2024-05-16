## Storage Classes
A StorageClass provides a way for administrators to describe the classes of storage they offer.

Different classes might map to quality-of-service levels, or to backup policies, or to arbitrary policies determined by the cluster administrators.

When a PVC does not specify a storageClassName, the default StorageClass is used.

### Create a Storage Class with NFS Driver
```bash
kubectl apply -f configs/storageclass.yaml
```

### Get the configured storage class information
```bash
kubectl get sc
```
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

### Dynamic Provisioning of PVs
Static provisioning of PVs, as shown above, is cumbersome: you must create the PV, and then the PVC while ensuring their properties match.

Dynamic provisioning is a more popular option for real-world use cases. The PV is created dynamically based on the PVC's configuration.

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
## CornJob

A CronJob creates Jobs on a repeating schedule.

### CronJob Spec
```
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of the month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday)
# │ │ │ │ │                                   OR sun, mon, tue, wed, thu, fri, sat
# │ │ │ │ │
# │ │ │ │ │
# * * * * *

For example, 0 0 13 * 5 states that the task must be started every Friday at midnight, as well as on the 13th of each month at midnight.
```
### Create a Cron Job
```bash
kubectl apply -f configs/cronjob.yaml
```

### Get the cron job details
```bash
kubectl get cronjob
```

### Get the logs from the Cron Job
```bash
kubectl logs jobs/k8s-labs-{replace-the-string-here} # You can submit to atomate it, PRs are welcome :).
```

## Job
A job creates one or more Pods and will continue to retry execution of the Pods until a specified number of them successfully terminate.

A simple case is to create one Job object in order to reliably run one Pod to completion. The Job object will start a new Pod if the first Pod fails or is delete.

### Run a job that computes π to 2000 places
```bash
kubectl apply -f configs/job.yaml
```

### List the pods associated with the job
```bash
pods=$(kubectl get pods --selector=batch.kubernetes.io/job-name=pi --output=jsonpath='{.items[*].metadata.name}')
```

### Check the logs of a pods
```bash
kubectl logs $pods
```

## Remove the Labs Resources
```bash
kubectl delete -f configs/
```
