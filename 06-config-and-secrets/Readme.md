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
kubectl apply -f configs/pod-configmap.yaml
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
kubectl apply -f configs/pod-secrets.yaml
```

#### Fetch the Secrets Value from a Pod
```bash
kubectl exec -it nginx -- /bin/sh -c 'echo $password'
```

