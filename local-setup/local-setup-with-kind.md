# K8s-labs: Local Setup

In this labs we will be using `kind` to provision a kubernetes cluster. Once the cluster is ready, We also require `kubectl` tool to interact with kubernetes cluster.

## kubectl
The kubernetes command-line tool, kubectl, allows you to run commands against kubernetes clusters. You can use kubectl to deploy applications, inspect and manage cluster resources, and view logs.
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
Kind lets you run kubernetes on your local computer. This tool requires that you have either Docker or Podman installed.
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

### Interacting with your Cluster
```bash
kind get clusters
kubectl cluster-info --context kind-k8s-labs
```

### Deleting a Cluster
```bash
kind delete cluster --name k8s-labs
```
