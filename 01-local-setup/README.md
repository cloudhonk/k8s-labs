# K8s-labs: Local Setup

In this labs we will be using `kind` to provision a kubernetes cluster. Once the cluster is ready, We also require `kubectl` tool to interact with kubernetes cluster.

## kubectl
The kubernetes command-line tool, kubectl, allows you to run commands against kubernetes clusters. You can use kubectl to deploy applications, inspect and manage cluster resources, and view logs. For more [details](https://kubernetes.io/docs/tasks/tools/#kubectl).
### Installation
#### On macOS
```bash
brew install kubectl
```
#### On Linux
1. Download the latest release
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```
2. Install kubectl
```bash
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```
3. Test to ensure the version you installed is up-to-date.
```bash
kubectl version --client
sudo chown root: /usr/local/bin/kubectl
```
### Kind
Kind lets you run kubernetes on your local computer. This tool requires that you have either Docker or Podman installed. For more [details](https://kind.sigs.k8s.io/docs/user/quick-start/#installation).
### Installation
#### On macOS
```bash
brew install kind
```
#### On Linux
```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

### Creating a Cluster
```bash
kind create cluster --name k8s-labs
```

### Creating a multi-node cluster
Previously, we had created a single node cluster. Now we will be setting up a multi-node cluster where we will have 1 master node and 2 worker nodes.

Copy the content locally with name `kind-config.yaml`

```bash
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
```

Now, use the local config file to create a multi-node cluster
```bash
kind create cluster --name k8s-labs --config kind-config.yaml
```

### Interacting with your Cluster
```bash
kind get clusters
```

```bash
kubectl cluster-info --context kind-k8s-labs
```

```bash
kubectl get nodes
```

### Deleting a Cluster
```bash
kind delete cluster --name k8s-labs
```
