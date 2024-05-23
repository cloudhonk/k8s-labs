## Kubernetes Network Model

- Every pod gets a unique cluster-wide ip address
- pods can communicate with all other pods on any other node without NAT
- agents on a node (e.g. system daemons, kubelet) can communicate with all pods on that node
- Kubernetes IP addresses exist at the Pod scope
  - containers within a Pod share their network namespaces - including their IP address and MAC address.
  - This means that containers within a Pod can all reach each other's ports on localhost.
- DNS:
  - Pod DNS: `<pod-ip>.<name-space>.pod.<cluster-domain>` eg: `10-244-0-21.default.pod.cluster.local`
  - Service DNS `<service-name>.<name-space>.pod.<cluster-domain>` eg: `kubernetes.default.svc.cluster.local`

## Kubernets IP Address Ranges

- Kubernetes clusters require to allocate non-overlapping IP addresses for Pods, Services and Nodes.
- The network plugin is configured to assign IP addresses to Pods.
- The kube-apiserver is configured to assign IP addresses to Services.
- The kubelet or the cloud-controller-manager is configured to assign IP addresses to Nodes.

## Service

In Kubernetes, a Service is a method for exposing a network application that is running as one or more Pods in your cluster.

![Service](../images/k8s-service.png)

### Service Demo
Deploy nginx config
```shell
kubectl apply -f configs/config.yaml
```

Deploy pods
```shell
~> kubectl apply -f configs/deployment.yaml 
deployment.apps/nginx created

~> kubectl get deployments.apps 
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   3/3     3            3           15s

~> k get pods -o wide -l=app=nginx
NAME                    READY   STATUS    RESTARTS   AGE   IP            NODE                     NOMINATED NODE   READINESS GATES
nginx-5f5c47659-4qpbd   1/1     Running   0          93s   10.244.0.27   k8s-labs-control-plane   <none>           <none>
nginx-5f5c47659-66854   1/1     Running   0          91s   10.244.0.28   k8s-labs-control-plane   <none>           <none>
nginx-5f5c47659-lbb4m   1/1     Running   0          90s   10.244.0.29   k8s-labs-control-plane   <none>           <none>
```

Deploy network utility pods
```shell
~> / # wget -qO- --server-response 10.244.0.27
  HTTP/1.1 200 OK
  Server: nginx/1.14.2
  Date: Thu, 23 May 2024 04:33:03 GMT
  Content-Type: text/html
  Content-Length: 612
  Last-Modified: Tue, 04 Dec 2018 14:44:49 GMT
  Connection: close
  ETag: "5c0692e1-264"
  Pod-Name: nginx-5f5c47659-4qpbd
  Accept-Ranges: bytes
  
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
/ # wget -qO- --server-response 10-244-0-27.default.pod.cluster.local
  HTTP/1.1 200 OK
  Server: nginx/1.14.2
  Date: Thu, 23 May 2024 04:34:08 GMT
  Content-Type: text/html
  Content-Length: 612
  Last-Modified: Tue, 04 Dec 2018 14:44:49 GMT
  Connection: close
  ETag: "5c0692e1-264"
  Pod-Name: nginx-5f5c47659-4qpbd
  Accept-Ranges: bytes
  
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

Create Service

```shell
~> kubectl apply -f configs/cluster-service.yaml
service/nginx created

~> kubectl get svc -o wide
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE   SELECTOR
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP   92m   <none>
nginx        ClusterIP   10.96.218.89   <none>        80/TCP    81s   app=nginx
```

Send Requests using the service

```shell
/ # wget -qO- --server-response http://nginx
  HTTP/1.1 200 OK
  Server: nginx/1.14.2
  Date: Thu, 23 May 2024 04:36:29 GMT
  Content-Type: text/html
  Content-Length: 612
  Last-Modified: Tue, 04 Dec 2018 14:44:49 GMT
  Connection: close
  ETag: "5c0692e1-264"
  Pod-Name: nginx-5f5c47659-4qpbd
  Accept-Ranges: bytes
  
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
/ # wget -qO- --server-response http://10.96.218.89 #Try This OR

/ # wget -qO- --server-response http://nginx.default.svc.cluster.local #Try this, it should also work
```
### Service Types

- Cluster IP(default): service will only be accessible inside the cluster, via the cluster IP.
- NodePort: Service will be exposed on one port of every node, in addition to 'ClusterIP' type.
- LoadBalancer: Service will be exposed via an external loadbalancer (if the cloud provider supports it), in addition to 'NodePort' type.
- ExternalName: Service consists of only a reference to an external name that kubedns or equivalent will return as a CNAME record.

NodePort Service

```shell
~> kubectl apply -f configs/nodeport-service.yaml
service/nodeport-nginx created

~> kubectl get svc -o wide
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE    SELECTOR
kubernetes       ClusterIP   10.96.0.1       <none>        443/TCP        151m   <none>
nginx            ClusterIP   10.96.218.89    <none>        80/TCP         60m    app=nginx
nodeport-nginx   NodePort    10.96.195.172   <none>        80:30187/TCP   93s    app=nginx

## Accessing from the Worker/ControlPlane Node
~> docker exec k8s-labs-control-plane curl localhost:30187
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   612  100   612    0     0  1147k      0 --:--:-- --:--:-- --:--:--  597k
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

Loadbalancer Service
```shell
~> kubectl apply -f configs/loadbalancer-service.yaml 
service/lb-nginx created

~> kubectl get svc -o wide
kubectl get svc -o wide
NAME             TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE     SELECTOR
kubernetes       ClusterIP      10.96.0.1       <none>        443/TCP        154m    <none>
lb-nginx         LoadBalancer   10.96.154.141   <pending>     80:31082/TCP   25s     app=nginx
nginx            ClusterIP      10.96.218.89    <none>        80/TCP         63m     app=nginx
nodeport-nginx   NodePort       10.96.195.172   <none>        80:30187/TCP   4m41s   app=nginx
```

ExternalName service

```shell
~> kubectl apply -f configs/externalname-service.yaml 
service/ext-service created

~> kubectl get svc -o wide
NAME             TYPE           CLUSTER-IP      EXTERNAL-IP       PORT(S)        AGE     SELECTOR
ext-service      ExternalName   <none>          httpforever.com   <none>         24s     <none>
kubernetes       ClusterIP      10.96.0.1       <none>            443/TCP        156m    <none>
lb-nginx         LoadBalancer   10.96.154.141   <pending>         80:31082/TCP   90s     app=nginx
nginx            ClusterIP      10.96.218.89    <none>            80/TCP         64m     app=nginx
nodeport-nginx   NodePort       10.96.195.172   <none>            80:30187/TCP   5m46s   app=nginx

~> kubectl run curl --image=appropriate/curl --restart=Never -- curl -sI http://ext-service
pod/curl created

~> kubectl logs curl
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Thu, 23 May 2024 06:02:26 GMT
Content-Type: text/html
Content-Length: 5124
Last-Modified: Wed, 22 Mar 2023 14:54:48 GMT
Connection: keep-alive
ETag: "641b16b8-1404"
Referrer-Policy: strict-origin-when-cross-origin
X-Content-Type-Options: nosniff
Feature-Policy: accelerometer 'none'; camera 'none'; geolocation 'none'; gyroscope 'none'; magnetometer 'none'; microphone 'none'; payment 'none'; usb 'none'
Content-Security-Policy: default-src 'self'; script-src cdnjs.cloudflare.com 'self'; style-src cdnjs.cloudflare.com 'self' fonts.googleapis.com 'unsafe-inline'; font-src fonts.googleapis.com fonts.gstatic.com cdnjs.cloudflare.com; frame-ancestors 'none'; report-uri https://scotthelme.report-uri.com/r/d/csp/enforce
Accept-Ranges: bytes

~> kubectl delete pod curl
```

## Ingress

Make your HTTP (or HTTPS) network service available using a protocol-aware configuration mechanism, that understands web concepts like URIs, hostnames, paths, and more. The Ingress concept lets you map traffic to different backends based on rules you define via the Kubernetes API.

## Ingress Controller

In order for an Ingress to work in your cluster, there must be an ingress controller running. You need to select at least one ingress controller and make sure it is set up in your cluster. This page lists common ingress controllers that you can deploy.

## Gateway API

Gateway API is a family of API kinds that provide dynamic infrastructure provisioning and advanced traffic routing.

