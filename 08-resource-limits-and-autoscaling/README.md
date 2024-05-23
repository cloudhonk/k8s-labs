# AutoScaling in K8s

## HorizontalPodAtuscaler Walkthrough
A HorizontalPodAutoscaler (HPA) automatically updates a workload resource(Deployment or StatefulSet)
with the aim of automatically scaling the workload to match the demand.

Horizontal Scaling: Deploy more Pods in response to increased load.
Vertical Scaling: Assigning more resources to the Pods that are already running.

## Lab Setup
In this labs, we will demonstrate a HPA in action where we will create a php-apache server. We will
create a Deployment that runs a container using the `hpa-example` image and expose it as a Service
using the manifests defined in `configs` directory.

### Apply the metric server resources to kind cluster
```bash
kubectl apply -f configs/components.yaml
```

### Run and Expose the server
```bash
kubectl apply -f configs/deployment.yaml
kubectl apply -f configs/service.yaml
```

### Create a HorizontalPodAutoscaler
```bash
kubectl autoscale deployment k8s-labs --cpu-percent=50 --min=1 --max=10
```

### Check the HPA Configuration
```bash
kubectl get hpa
```

## Increase the load to PHP Apache Server
Now we are going to observe how the autoscaler reacts to increased load. In order to increase the load
we will run a infinite loop sending queries to the php-apache server

```bash
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://k8s-labs; done"
```

In another terminal, execute the below command to watch how Autoscaler act on the HPA.
```bash
kubectl get hpa k8s-labs --watch
```

Within a minute or so, you should see the higher CPU load as shown:
```
NAME       REFERENCE             TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
k8s-labs   Deployment/k8s-labs   150%/50%    1         10        1          71m
```

After a while, you would see more replicas being created
```
NAME       REFERENCE             TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
k8s-labs   Deployment/k8s-labs   155%/50%    1         10        3        75m
```

Watch the deployment state to match with HPA

```bash
kubectl get deployment k8s-labs
```

## Stop the load on the cluster
In the terminal where you created the Pod that runs a `busybox` image, terminate the load generating by executing `<CTRL + C`

After a minute or so you can verify the result state where replica is reduced down to 1.
```bash
kubectl get hpa k8s-labs --watch
```

```
NAME       REFERENCE             TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
k8s-labs   Deployment/k8s-labs   0%/50%    1         10        1          80m
```
