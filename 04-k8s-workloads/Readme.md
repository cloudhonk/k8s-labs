### Pod

Pods are the smallest deployable units of computing that you can create and manage in Kubernetes.
It is a group of one or more containers, with shared storage and network resources, and a specification for how to run the containers.

#### Few Points about Pods
- Pods are ephimeral and disposable entities
- Usually we don't create pods directly, we create them using workload resources like Deployments or job. If the pods require to have a statefullness, consider using Statefulset resource.
- Each pod is meant to run single instance of a given application.

#### Phases of Pod
- Pending
- Running
- Succeeded
- Failed
- Unknown

#### Create a Pod
```bash
kubectl apply -f configs/pod.yaml
```

#### Delete a running Pod
```bash
kubectl delete -f configs/pod.yaml
```

[Read more on Pods](https://kubernetes.io/docs/concepts/workloads/pods/)

### Deployments

A Deployment provides declarative updates for Pods and ReplicaSets.

#### Use Cases
- Declare the new state of the Pods
- Rollback to an earlier deployment revision
- Scale up the deployment to facilitate more load
- Pause the rollout of a deployment
- Create a deployment to rollout a replicaset
- Use the status of the deployment
- Clean up older ReplicaSets

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

### Service

A Service is a method for exposing a network application that is running as one or more Pods in your cluster.

#### Service Types
- ClusterIP: Default service type, it exposes the service on an internal IP address within the cluster and can only be accessible from within the cluster.
- NodePort: Exposes the Service on a port across all nodes in the custer(Port ranges: 30000-32767)
- LoadBalancer: Provision external loadbalancer to distribute traffic to the service. Useful for exposing service to the clients.

#### Create a Service
```bash
kubectl apply -f configs/service.yaml
```

#### Delete a Service
```bash
kubectl delete -f conifigs/service.yaml
```

### ConfigMap
A ConfigMap is an API object used to store non-confidential data in key-value pairs. Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume.

#### Create a ConfigMaps
```bash
kubectl apply -f configs/configmap.yaml
```

#### Get the ConfigMaps
```bash
kubectl describe configmaps k8s-labs
```

#### Reference ConfigMap with a Pod
```bash
kubectl apply -f configs/pod.yaml
```

#### Fetch the values of the ConfigMaps from Pod
```bash
kubectl exec -it nginx -- env | grep -E 'LOG_LEVEL|API_KEY|LAB_NAME'
```

### Secrets
A Secret is an object that contains a small amount of sensitive data such as a password, a token, or a key.

#### Create a secrets
```bash
kubectl apply -f configs/secrets.yaml
```

#### Configure Pods with Secrets
```bash
kubectl apply -f configs/pod.yaml
```

#### Fetch the Secrets Value from a Pod
```bash
kubectl exec -it nginx -- /bin/sh -c 'echo $password'
```

