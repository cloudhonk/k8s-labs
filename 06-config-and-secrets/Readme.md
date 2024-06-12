### ConfigMap
A ConfigMap is an API object used to store non-confidential data in key-value pairs. Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume.

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

### Consuming ConfigMap

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
~> kubectl apply -f configs/configmap-env-pod.yaml
pod/configmap-env-pod created

~> kubectl exec configmap-env-pod -- env | grep -e 'PLAYER_INITIAL_LIVES' -e 'UI_PROPERTIES_FILE_NAME'
PLAYER_INITIAL_LIVES=3
UI_PROPERTIES_FILE_NAME=user-interface.properties
```

Furthermore, we can associate ConfigMaps with Deployment resource as well.
```bash
kubectl apply -f ./configs/configmap-deployment.yaml
```

```bash
kubectl exec deployment/k8s-labs-deployment -- env | grep -e 'LAB_NAME' -e 'LOG_LEVEL'
```

**as volume mount**

- Mounted ConfigMaps are updated automatically
- A container using a ConfigMap as a subPath volume mount will not receive ConfigMap updates.
- The kubelet checks whether the mounted ConfigMap is fresh on every periodic sync. However, the kubelet uses its local cache for getting the current value of the ConfigMap. The type of the cache is configurable using the `configMapAndSecretChangeDetectionStrategy` field in the KubeletConfiguration struct.
- A ConfigMap can be either propagated by watch (default), ttl-based, or by redirecting all requests directly to the API server from kubelet config.
- Because existing Pods maintain a mount point to the deleted ConfigMap, it is recommended to recreate these pods.

Deploy the volume consumer pod
```bash
~> kubectl apply -f configs/configmap-volume-pod.yaml
pod/configmap-volume-pod created

~> kubectl exec configmap-volume-pod -- env | grep -e 'PLAYER_INITIAL_LIVES' -e 'UI_PROPERTIES_FILE_NAME'

~> kubectl exec configmap-volume-pod -- ls /config
game.properties
user-interface.properties

~> kubectl exec configmap-volume-pod -- cat /config/game.properties
enemy.types=aliens,monsters
player.maximum-lives=5
```

**as Custom Initialization**

ConfigMaps can be used in init containers to set up an environment before the main application starts. Here we will be using `InitContainer` concepts to spin up helper container that will run first and do some stuff before running the actual application container.

The InitContainer utilize the configmaps script and run it before the application starts.

### Apply the ConfigMap and Pod configuration
```bash
kubectl apply -f configs/configmap-init-container.yaml
configmap/init-config created
pod/k8s-labs-01 created
```

### Check the logs of Init Containers
```bash
kubectl logs k8s-labs-01 -c k8s-labs-initcontainer
Setting up environment
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

NOTES:
- Secrets are similar to ConfigMaps but are specifically intended to hold confidential data.
- Kubernetes Secrets are, by default, stored unencrypted in the API server's underlying data store (etcd). Anyone with API access can retrieve or modify a Secret, and so can anyone with access to etcd.
- In order to safely use Secrets, take at least the following steps:
  - Enable Encryption at Rest for Secrets.
  - Enable or configure RBAC rules with least-privilege access to Secrets.
  - Restrict Secret access to specific containers.
  - Consider using external Secret store providers.

### Secret Types

#### 1. Opaque: arbitrary user-defined data
Opaque is the default Secret type if you don't explicitly specify a type in a Secret manifest.

```shell
kubectl create secret generic empty-secret
kubectl get secret empty-secret
```

NOTES:

- You can define and use your own Secret type by assigning a non-empty string as the type value for a Secret object (an empty string is treated as an Opaque type).

- Kubernetes doesn't impose any constraints on the type name. However, if you are using one of the built-in types, you must meet all the requirements defined for that type.
- **kubernetes.io/service-account-token**: ServiceAccount token

A `kubernetes.io/service-account-token` type of Secret is used to store a token credential that identifies a ServiceAccount. This is a `legacy mechanism that provides long-lived ServiceAccount credentials` to Pods.

In Kubernetes v1.22 and later, the recommended approach is to obtain a short-lived, automatically rotating ServiceAccount token by using the TokenRequest API instead. You can get these short-lived tokens using the following methods:

- Call the TokenRequest API either directly or by using an API client like kubectl. eg. `kubectl create token command`.
- Request a mounted token in a projected volume in your Pod manifest. Kubernetes creates the token and mounts it in the Pod.

```yaml
k apply -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: null
  name: sample-sa
---kubectl get secret secret-tiger-docker -o jsonpath='{.data.*}' | base64 -d

apiVersion: v1
kind: Secret
metadata:
  name: service-account-secret
  annotations:
    kubernetes.io/service-account.name: "sample-sa"
type: kubernetes.io/service-account-token
data:
  extra: YmFyCg==
EOF
```

#### 2. Docker Registry Secret
  - SubType: `kubernetes.io/dockercfg`
    This is legacy type. Should be base64 encoded content of the `~/.dockercfg` file.
  - SubType: `kubernetes.io/dockerconfigjson`
    This is recommended latest type. Should be base64 encoded of the `~/.docker/config.json` file

examples
```shell
kubectl create secret docker-registry secret-tiger-docker \
  --docker-email=tiger@acme.example \
  --docker-username=tiger \
  --docker-password=pass1234 \
  --docker-server=my-registry.example:5000

kubectl get secret secret-tiger-docker -o jsonpath='{.data.*}' | base64 -d | jq

{
  "auths": {
    "my-registry.example:5000": {
      "username": "tiger",
      "password": "pass1234",
      "email": "tiger@acme.example",
      "auth": "dGlnZXI6cGFzczEyMzQ="
    }
  }
}

kubectl delete secret secret-tiger-docker
```

NOTE: It is suggested to use [credential providers](https://kubernetes.io/docs/tasks/administer-cluster/kubelet-credential-provider/) to dynamically and securely provide pull secrets on-demand.

#### 3. kubernetes.io/basic-auth: credentials for basic authentication
When using this Secret type, the data field of the Secret must contain one of the following two keys:

`username`: the user name for authentication
`password`: the password or token for authentication

```yaml
kubectl apply --dry-run=server -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: secret-basic-auth
type: kubernetes.io/basic-auth
stringData:
  username: admin # required field for kubernetes.io/basic-auth
  password: t0p-Secret # required field for kubernetes.io/basic-auth
EOF
```
#### 4. kubernetes.io/ssh-auth: credentials for SSH authentication

When using this Secret type, you will have to specify a `ssh-privatekey` key-value pair in the data (or stringData) field as the SSH credential to use.

```shell
kubectl apply --dry-run=server -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: secret-ssh-auth
type: kubernetes.io/ssh-auth
data:
  # the data is abbreviated in this example
  ssh-privatekey: |
    UG91cmluZzYlRW1vdGljb24lU2N1YmE=
```

#### 5. kubernetes.io/tls: for a TLS client or server
Secret type is for storing a certificate and its associated key that are typically used for TLS. When using this type of Secret, the `tls.key` and the `tls.crt` key must be provided in the data (or stringData) field of the Secret configuration

```shell
## using manifest file
kubectl apply --dry-run=server -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: secret-tls
type: kubernetes.io/tls
data:
  # values are base64 encoded, which obscures them but does NOT provide
  # any useful level of confidentiality
  tls.crt: |
    LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNVakNDQWJzQ0FnMytNQTBHQ1NxR1NJYjNE
    UUVCQlFVQU1JR2JNUXN3Q1FZRFZRUUdFd0pLVURFT01Bd0cKQTFVRUNCTUZWRzlyZVc4eEVEQU9C
    Z05WQkFjVEIwTm9kVzh0YTNVeEVUQVBCZ05WQkFvVENFWnlZVzVyTkVSRQpNUmd3RmdZRFZRUUxF
    dzlYWldKRFpYSjBJRk4xY0hCdmNuUXhHREFXQmdOVkJBTVREMFp5WVc1ck5FUkVJRmRsCllpQkRR
    VEVqTUNFR0NTcUdTSWIzRFFFSkFSWVVjM1Z3Y0c5eWRFQm1jbUZ1YXpSa1pDNWpiMjB3SGhjTk1U
    TXcKTVRFeE1EUTFNVE01V2hjTk1UZ3dNVEV3TURRMU1UTTVXakJMTVFzd0NRWURWUVFHREFKS1VE
    RVBNQTBHQTFVRQpDQXdHWEZSdmEzbHZNUkV3RHdZRFZRUUtEQWhHY21GdWF6UkVSREVZTUJZR0Ex
    VUVBd3dQZDNkM0xtVjRZVzF3CmJHVXVZMjl0TUlHYU1BMEdDU3FHU0liM0RRRUJBUVVBQTRHSUFE
    Q0JoQUo5WThFaUhmeHhNL25PbjJTbkkxWHgKRHdPdEJEVDFKRjBReTliMVlKanV2YjdjaTEwZjVN
    Vm1UQllqMUZTVWZNOU1vejJDVVFZdW4yRFljV29IcFA4ZQpqSG1BUFVrNVd5cDJRN1ArMjh1bklI
    QkphVGZlQ09PekZSUFY2MEdTWWUzNmFScG04L3dVVm16eGFLOGtCOWVaCmhPN3F1TjdtSWQxL2pW
    cTNKODhDQXdFQUFUQU5CZ2txaGtpRzl3MEJBUVVGQUFPQmdRQU1meTQzeE15OHh3QTUKVjF2T2NS
    OEtyNWNaSXdtbFhCUU8xeFEzazlxSGtyNFlUY1JxTVQ5WjVKTm1rWHYxK2VSaGcwTi9WMW5NUTRZ
    RgpnWXcxbnlESnBnOTduZUV4VzQyeXVlMFlHSDYyV1hYUUhyOVNVREgrRlowVnQvRGZsdklVTWRj
    UUFEZjM4aU9zCjlQbG1kb3YrcE0vNCs5a1h5aDhSUEkzZXZ6OS9NQT09Ci0tLS0tRU5EIENFUlRJ
    RklDQVRFLS0tLS0K    
  # In this example, the key data is not a real PEM-encoded private key
  tls.key: |
    RXhhbXBsZSBkYXRhIGZvciB0aGUgVExTIGNydCBmaWVsZA==
EOF

### Using Kubectl
kubectl create secret tls my-tls-secret \
  --cert=path/to/cert/file \
  --key=path/to/key/file
```

#### 6. bootstrap.kubernetes.io/token:	bootstrap token data

Secret type is for tokens used during the node bootstrap process. It stores tokens used to sign well-known ConfigMaps. A bootstrap token Secret is usually created in the kube-system namespace and named in the form bootstrap-token-<token-id> where <token-id> is a 6 character string of the token ID.

Example
```shell
kubectl apply --dry-run=server -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: bootstrap-token-5emitj
  namespace: kube-system
type: bootstrap.kubernetes.io/token
data:
  auth-extra-groups: c3lzdGVtOmJvb3RzdHJhcHBlcnM6a3ViZWFkbTpkZWZhdWx0LW5vZGUtdG9rZW4=
  expiration: MjAyMC0wOS0xM1QwNDozOToxMFo=
  token-id: NWVtaXRq
  token-secret: a3E0Z2lodnN6emduMXAwcg==
  usage-bootstrap-authentication: dHJ1ZQ==
  usage-bootstrap-signing: dHJ1ZQ==
EOF
```

### Consuming Secret

- Secret needs to be created before any Pods that depend on it.
- When you reference a Secret in a Pod, you can mark the Secret as optional.
```yaml
....
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      optional: true
```
- When a volume contains data from a Secret, and that Secret is updated, Kubernetes tracks this and updates the data in the volume, using an eventually-consistent approach.
- A container using a Secret as a `subPath volume mount` does not receive automated Secret updates.
- You cannot use ConfigMaps or Secrets with `static Pods`.
- You can create an immutable Secret by setting the `immutable field to true`.
- A Secret is only sent to a node if a Pod on that node requires it. 
- For mounting Secrets into Pods, the kubelet stores a copy of the data into a tmpfs so that the confidential data is not written to durable storage. 
- You must explicitly define environment variables or map a volume into a container in order to provide access to any other Secret.

#### Create a secrets

```bash
kubectl apply -f configs/secrets.yaml
```

#### Consume with env var
```bash
kubectl apply -f configs/secrets-env-pod.yaml

kubectl exec secret-env-pod -- /bin/sh -c 'echo $password'
```

#### Consume with volume

```bash
kubectl apply -f configs/secrets-volume-pod.yaml

kubectl exec secret-volume-pod -- ls -la /secrets
```
