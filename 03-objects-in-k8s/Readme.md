## Understanding Objects in K8s [link](https://kubernetes.io/docs/concepts/overview/working-with-objects/)

- A Kubernetes object is a "record of intent"--once you create the object, the Kubernetes system will constantly work to ensure that object exists.

```shell
kubect run nginx --image=nginx 
```
- Kubernetes objects are persistent entities in the Kubernetes system.
- Kubernetes uses these entities to represent the state of your cluster. 
- Kubernetes objects are managed using the Kubernetes API.

**Object Configuration**

- Almost every Kubernetes object the object `spec` and the object `status` config. 
- you have to set `spec` when you create the object, providing the desired state.
- The `status` describes the current state of the object.

**Required Fields**

- `apiVersion`, `kind`, `metadata`, `spec`

### 1. Object Management [link](https://kubernetes.io/docs/concepts/overview/working-with-objects/object-management/)

- kubectl imperative
- kubectl imperative using config files
- kubectl declarative using config files

### 2. Object Names and IDs [link](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/)

Each object in your cluster has a Name that is unique for that type of resource. Every Kubernetes object also has a UID that is unique across your whole cluster.

### 3. Namespaces [link](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

In Kubernetes, namespaces provide a mechanism for isolating groups of resources within a single cluster. Names of resources need to be unique within a namespace, but not across namespaces.

- Names of resources need to be unique within a namespace, but not across namespaces.
- Namespaces are a way to divide cluster resources between multiple users (via resource quota).
- Namespaces are intended for use in environments with many users spread across multiple teams, or projects. 
- For clusters with a few to tens of users, you should not need to create or think about namespaces at all. 

**Initial Namespaces**

- `default`, `kube-node-lease`, `kube-public`, `kube-system`

