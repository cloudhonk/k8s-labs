# Troubleshooting Apps in Kubernetes

You'll have to spend a lot of the time in Kubectl troubleshooting problems.

## K8s-Labs

Please revise the deployment and service classes. The problem statement is only based on these resources.
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
