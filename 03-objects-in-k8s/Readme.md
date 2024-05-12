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
It is a group of one or more containers, with shared storage and entwork resources, and a specification for how to run the containers.

#### Few Points about pods
- Pods are ephimeral and disposable entities
- Usually we don't create pods directly, we create them using workload resources like Deployments or job. If the pods require to have a statefullness, consider using Statefulset resource.
- Each pod is meant to run single instance of a given application.

```bash
kubectl apply -f configs/pod.yaml
```
