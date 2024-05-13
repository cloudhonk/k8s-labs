### Understanding Objects in K8s
- A Kubernetes object is a `record of intent` once you create the object, the Kubernetes system will constantly work to ensure that object exists.
- Kubernetes objects are persistent entities in the Kubernetes system.
- Kubernetes uses these entities to represent the state of your cluster.
- Kubernetes objects are managed using the Kubernetes API.

    [Read more on kubernetes objects](https://kubernetes.io/docs/concepts/overview/working-with-objects/)

### Object Configuration
- Almost every Kubernetes object the object `spec` and the object `status` config.
- You have to set `spec` when you create the object, providing the desired state.
- The `status` describes the current state of the object.

**Required Fields**

- `apiVersion`, `kind`, `metadata`, `spec`

### Object Management

#### Imperative commands


Each object in your cluster has a Name that is unique for that type of resource. Every Kubernetes object also has a UID that is unique across your whole cluster.

```bash
kubectl run nginx --image nginx
kubectl port-forward nginx 8989:80
kubectl delete pod nginx
```

#### Imperative Object Configuration

```bash
kubectl create -f configs/pod.yaml
```

```bash
kubectl delete -f configs/pod.yaml
```

#### Declarative Object Configuration

```bash
kubectl apply -f configs/
```

```bash
kubectl diff -f configs/
```


```bash
kubectl delete -f configs/
```

[Read more on Object Management](https://kubernetes.io/docs/concepts/overview/working-with-objects/object-management/)

### Object Names and IDs

Each object in your cluster has a Name that is unique for that type of resource. Every Kubernetes object also has a UID that is unique across your whole cluster.

[Read more on Object Names and IDs](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/)

### Namespace

In Kubernetes, namespaces provide a mechanism for isolating groups of resources within a single cluster. Names of resources need to be unique within a namespace, but not across namespaces.

- Names of resources need to be unique within a namespace, but not across namespaces.
- Namespaces are a way to divide cluster resources between multiple users (via resource quota).
- Namespaces are intended for use in environments with many users spread across multiple teams, or projects.
- For clusters with a few to tens of users, you should not need to create or think about namespaces at all.

**Initial Namespaces**
```bash
kubectl get namespaces
```
- `default`, `kube-node-lease`, `kube-public`, `kube-system`

    [Read more on Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

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

