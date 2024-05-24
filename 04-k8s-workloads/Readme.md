### Pod

Pods are the smallest deployable units of computing that you can create and manage in Kubernetes.
It is a group of one or more containers, with shared storage and network resources, and a specification for how to run the containers.

![Pod](../images/pea.png)

#### Few Points about Pods
- Pods are ephimeral and disposable entities
- Pods are immutable in nature. 
- Pod has its own IP address. Containers Inside a pod share the same network and IP. 
- Usually we don't create pods directly, we create them using workload resources like Deployments or job. If the pods require to have a statefullness, consider using Statefulset resource.
- Each pod is meant to run single instance of a given application.

![Pod](../images/pod.png)

#### Phases of Pod
- Pending
- Running
- Succeeded
- Failed
- Unknown

#### Create a Pod

**Example**

Creating via kubectl

```shell
kubectl run sleeping-pod --image=busybox --restart=Never --command -- sleep 5
```

**Example**

Basic pod template

```yml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mycontainer
    image: nginx
```

**Example**

Adding Environment Variable

```yml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mycontainer
    image: nginx
    env:
    - name: MY_VAR
      value: "hello"
```

**Example**

Providing resource contraints
```yml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mycontainer
    image: nginx
    resources:
      limits:
        cpu: "0.5"
        memory: "512Mi"
      requests:
        cpu: "0.25"
        memory: "256Mi"
```

[Read more on Pods](https://kubernetes.io/docs/concepts/workloads/pods/)

### Deployments

A Deployment provides declarative updates for Pods and ReplicaSets.

![Pod](../images/deployment.png)

#### Use Cases
- Declare the new state of the Pods
- Rollback to an earlier deployment revision
- Scale up the deployment to facilitate more load
- Pause the rollout of a deployment
- Create a deployment to rollout a replicaset
- Use the status of the deployment
- Clean up older ReplicaSets

**Example**

**Initial Deployment**

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mydeployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: mycontainer
        image: nginx
        ports:
        - containerPort: 80

```

**Adding Environment Variables**

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mydeployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: mycontainer
        image: nginx
        ports:
        - containerPort: 80
        env:
        - name: MY_VAR
          value: "hello"

```

#### Create a Deployment
```bash
kubectl apply -f configs/deployment.yaml
```

#### Get the Deployments
```bash
kubectl get deployments
```

#### See the Deployment Rollout status
```bash
kubectl rollout status deployment/nginx-deployment
```

#### Delete the Deployments
```bash
kubectl delete -f configs/deployment.yaml
```

#### Other information on Deployments

- https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment
- https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-back-a-deployment

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