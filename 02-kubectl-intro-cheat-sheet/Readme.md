## Resource link

### 1. kubectl and configs

- kubectl config default location is `~/.kube/config`
- viewing the config
```shell
-> kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://127.0.0.1:6443
  name: docker-desktop
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://127.0.0.1:50445
  name: kind-kind
contexts:
- context:
    cluster: docker-desktop
    user: docker-desktop
  name: docker-desktop
- context:
    cluster: kind-kind
    user: kind-kind
  name: kind-kind
current-context: kind-kind
kind: Config
preferences: {}
users:
- name: docker-desktop
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
- name: kind-kind
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
```
- config contains three major components
  - clusters
  - contexts
  - users

- list contexts
```shell
-> k config get-contexts
CURRENT   NAME             CLUSTER          AUTHINFO         NAMESPACE
          docker-desktop   docker-desktop   docker-desktop   
*         kind-kind        kind-kind        kind-kind
```
- switch another context
```shell
-> k config use-context docker-desktop
Switched to context "docker-desktop".

-> k config get-contexts
CURRENT   NAME             CLUSTER          AUTHINFO         NAMESPACE
*         docker-desktop   docker-desktop   docker-desktop   
          kind-kind        kind-kind        kind-kind

-> k config use-context kind-kind
```

### 2. Kubectl intro for docker users

https://kubernetes.io/docs/reference/kubectl/docker-cli-to-kubectl/

- **docker run** and **kubectl run**
```shell
-> docker run -d --restart=always -e DOMAIN=cluster -p9099:80 nginx

1891995ed4a28854bab95edf9a538303f45ccf4cc57fe86c94287a8735d2c846

-> docker ps

CONTAINER ID   IMAGE      COMMAND                  CREATED         STATUS          PORTS                       NAMES
1891995ed4a2   nginx      "/docker-entrypoint.…"   5 minutes ago   Up 5 minutes   0.0.0.0:9099->80/tcp         nervous_hoover


-> kubectl create deployment --image=nginx nginx-app
deployment.apps/nginx-app created

-> kubectl set env deployment/nginx-app  DOMAIN=cluster
deployment.apps/nginx-app env updated

-> kubectl get pod
NAME                         READY   STATUS    RESTARTS   AGE
nginx-app-5c977c4f9c-cndv5   1/1     Running   0          93s

-> kubectl get deployment
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
nginx-app   1/1     1            1           2m34s
```

- **docker attach** and **kubectl attach**

```shell
-> docker ps

CONTAINER ID   IMAGE      COMMAND                  CREATED         STATUS          PORTS                       NAMES
1891995ed4a2   nginx      "/docker-entrypoint.…"   5 minutes ago   Up 5 minutes   0.0.0.0:9099->80/tcp         nervous_hoover

-> docker attach 1891995ed4a2
192.168.65.1 - - [09/May/2024:02:56:15 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.88.1" "-"

-> kubectl get pod
NAME                         READY   STATUS    RESTARTS   AGE
nginx-app-5c977c4f9c-cndv5   1/1     Running   0          6m3s

-> kubectl attach -it nginx-app-5c977c4f9c-cndv5
127.0.0.1 - - [09/May/2024:02:59:13 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.88.1" "-"

```

- **docker exec** and **kubectl exec**

```shell
-> docker ps
CONTAINER ID   IMAGE      COMMAND                  CREATED         STATUS          PORTS                       NAMES
1891995ed4a2   nginx      "/docker-entrypoint.…"   5 minutes ago   Up 5 minutes   0.0.0.0:9099->80/tcp         nervous_hoover

## running the command
-> docker exec 1891995ed4a2 cat /etc/hostname
1891995ed4a2

## opening the shell
-> docker exec -it 1891995ed4a2 bash
root@1891995ed4a2:/# 

-> kubectl get pod
NAME                         READY   STATUS    RESTARTS   AGE
nginx-app-5c977c4f9c-cndv5   1/1     Running   0          6m3s

## running the command
-> kubectl exec nginx-app-5c977c4f9c-cndv5 -- cat /etc/hostname
nginx-app-5c977c4f9c-cndv5

## opening the shell
-> kubectl exec -it nginx-app-5c977c4f9c-cndv5 -- bash
root@nginx-app-5c977c4f9c-cndv5:/# 
```

- **docker cp** and **kubectl cp**
```shell
-> docker ps

CONTAINER ID   IMAGE      COMMAND                  CREATED         STATUS          PORTS                       NAMES
1891995ed4a2   nginx      "/docker-entrypoint.…"   5 minutes ago   Up 5 minutes   0.0.0.0:9099->80/tcp         nervous_hoover

-> docker cp 1891995ed4a2:/etc/hostname ./container-hostname
Successfully copied 2.05kB to /Users/cloudhonk/personal/github/cloudhonk/k8s-labs/container-hostname

-> kubectl get pod
NAME                         READY   STATUS    RESTARTS   AGE
nginx-app-5c977c4f9c-cndv5   1/1     Running   0          6m3s

-> kubectl cp nginx-app-5c977c4f9c-cndv5:etc/hostname ./pod-hostname
```

- **docker stop/rm** and **kubectl delete**

```shell
-> docker ps

CONTAINER ID   IMAGE      COMMAND                  CREATED         STATUS          PORTS                       NAMES
1891995ed4a2   nginx      "/docker-entrypoint.…"   5 minutes ago   Up 5 minutes   0.0.0.0:9099->80/tcp         nervous_hoover

-> docker stop 1891995ed4a2 && docker rm 1891995ed4a2
1891995ed4a2
1891995ed4a2

-> docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED       STATUS      PORTS                       NAMES

-> kubectl get deployment
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
nginx-app   1/1     1            1           17m

-> kubectl delete deployment nginx-app
deployment.apps "nginx-app" deleted

-> kubectl get deployment
No resources found in default namespace.

-> kubectl get pod
No resources found in default namespace.
```

- **docker info** and **kubectl cluster-info**

```shell
-> docker info
Client:
 Version:    25.0.2
 Context:    desktop-linux
 Debug Mode: false
 Plugins:
  buildx: Docker Buildx (Docker Inc.)
    Version:  v0.12.1-desktop.4
    Path:     /Users/cloudhonk/.docker/cli-plugins/docker-buildx
  compose: Docker Compose (Docker Inc.)
    Version:  v2.24.3-desktop.1
    ....
-> kubectl cluster-info
Kubernetes control plane is running at https://127.0.0.1:50445
CoreDNS is running at https://127.0.0.1:50445/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

### 3. Quick Reference
https://kubernetes.io/docs/reference/kubectl/quick-reference/

### 4. Kubectl Commands
https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands

### 5. Kubectl Reference
https://kubernetes.io/docs/reference/kubectl/generated/