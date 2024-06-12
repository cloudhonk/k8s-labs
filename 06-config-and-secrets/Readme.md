### ConfigMap
A ConfigMap is an API object used to store non-confidential data in key-value pairs. Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume.

[ConfigMap](../images/config-map.png)

**NOTES:**
- ConfigMap object is not designed to hold large chunks of data. The data stored in a ConfigMap cannot exceed 1 MiB.
- ConfigMap object has data and binaryData fields. The data field is designed to contain UTF-8 strings while the binaryData field is designed to contain binary data as base64-encoded strings.
- The name of a ConfigMap must be a valid DNS subdomain name.
- Each key under the data or the binaryData field must consist of alphanumeric characters, -, _ or ..
- The keys stored in data must not overlap with the keys in the binaryData field.
- Starting from v1.19, you can add an immutable field to a ConfigMap definition to create an immutable. [Details](https://kubernetes.io/docs/concepts/configuration/configmap/#configmap-immutable).

There are four different ways that you can use a ConfigMap to configure a container inside a Pod:

1. Inside a container command and args
2. Environment variables for a container
3. Add a file in read-only volume, for the application to read
4. Write code to run inside the Pod that uses the Kubernetes API to read a ConfigMap

### Consume ConfigMap

Create a ConfigMaps
```bash
~> kubectl apply -f configs/configmap.yaml
configmap/game-demo created

~> kubectl get configmap
NAME               DATA   AGE
game-demo          4      37s
kube-root-ca.crt   1      6d2h

~> kubectl describe configmap game-demo
Name:         game-demo
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
user-interface.properties:
----
color.good=purple
color.bad=yellow
allow.textmode=true

game.properties:
----
enemy.types=aliens,monsters
player.maximum-lives=5

player_initial_lives:
----
3
ui_properties_file_name:
----
user-interface.properties

BinaryData
====

Events:  <none>
```

**as environment variables**
- ConfigMaps consumed as environment variables are not updated automatically and require a pod restart.
- It's important to note that the range of characters allowed for environment variable names in pods is [restricted](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/#using-environment-variables-inside-of-your-config). If any keys do not meet the rules, those keys are not made available to your container, though the Pod is allowed to start.

Deploy the envvar consumer pod
```bash
~> kubectl apply -f configs/pod-envvar.yaml
pod/configmap-env-pod created

~> kubectl exec configmap-env-pod -- env | grep -e 'PLAYER_INITIAL_LIVES' -e 'UI_PROPERTIES_FILE_NAME'
PLAYER_INITIAL_LIVES=3
UI_PROPERTIES_FILE_NAME=user-interface.properties
```

**as volume mount**

- Mounted ConfigMaps are updated automatically
- A container using a ConfigMap as a subPath volume mount will not receive ConfigMap updates.
- The kubelet checks whether the mounted ConfigMap is fresh on every periodic sync. However, the kubelet uses its local cache for getting the current value of the ConfigMap. The type of the cache is configurable using the `configMapAndSecretChangeDetectionStrategy` field in the KubeletConfiguration struct.
- A ConfigMap can be either propagated by watch (default), ttl-based, or by redirecting all requests directly to the API server from kubelet config.
- Because existing Pods maintain a mount point to the deleted ConfigMap, it is recommended to recreate these pods.

Deploy the volume consumer pod
```bash
~> kubectl apply -f configs/pod-volume.yaml
pod/configmap-volume-pod created

~> kubectl exec configmap-volume-pod -- env | grep -e 'PLAYER_INITIAL_LIVES' -e 'UI_PROPERTIES_FILE_NAME'

~> kubectl exec -it configmap-volume-pod -- /bin/sh
```

**as Custom Initialization

ConfigMaps can be used in init containers to set up an environment before the main application starts. Here we will be using `InitContainer` concepts to spin up helper container that will run first and do some stuff before running the actual application container.

The InitContainer utilize the configmaps script and run it before the application starts.

### Apply the ConfigMap and Pod configuration
```bash
kubectl apply -f configs/cm-pod-init.yaml
```

### Check the logs of Init Containers
```bash
kubectl logs k8s-labs-01 -c k8s-labs-initcontainer
```

### Immutable Configmap/Secrets

- Once a ConfigMap is marked as immutable, You can only delete and recreate the ConfigMap
- For clusters that extensively use ConfigMaps, preventing changes to their data has the following advantages:
  - protects you from accidental (or unwanted) updates that could cause applications outages
  - improves performance of your cluster by significantly reducing load on kube-apiserver, by closing watches for ConfigMaps marked as immutable.

You can create an immutable ConfigMap by setting the immutable field to true. For example:

```shell
apiVersion: v1
kind: ConfigMap
metadata:
  ...
data:
  ...
immutable: true
```

### Secrets
A Secret is an object that contains a small amount of sensitive data such as a password, a token, or a key.

[Secrets](../images/K8s-Secret.png)

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
kubectl exec -it nginx-101 -- /bin/sh -c 'echo $password'
```
