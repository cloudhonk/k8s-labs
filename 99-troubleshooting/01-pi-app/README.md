# k8s-labs

### Prerequisites:
- Create a cluster with kind configuration located at `01-local-setup`
```bash
kind create cluster --name k8s-labs --config ../../01-local-setup/kind-config.yaml
```
- Revise the deployment and service classes

### Problem Statement
Currently the deployment and services are in broken state. You can make whatever changes you need to get the application running.

```
kubectl apply -f configs/
```

> Expectation: You should be able to browse the PI app at `http://localhost:8020` or http://localhost:30000 and see the response from the Pi app

___
## Cleanup

When you're done you can remove all the objects:

```
kubectl delete all -l app=k8s-labs
```
