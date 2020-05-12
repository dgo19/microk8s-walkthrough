# microk8s walkthrough

## Requirements
- ubuntu or linux distributrion with snap
- addition packages
  - curl

## Installation of microk8s
```
$ snap install microk8s --classic
microk8s v1.18.2 from Canonical installed
```
Add user to microk8s group and set alias for kubectl
```
$ sudo usermod -a -G microk8s dgo
$ echo "alias kubectl='microk8s kubectl'" >> .bashrc
$ echo 'source <(microk8s kubectl completion bash)' >> .bashrc
```
Get kustomize
```
$ mkdir ~/bin
$ curl -L https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize%2Fv3.5.4/kustomize_v3.5.4_linux_amd64.tar.gz | tar xvzC ~/bin -f -
```
Re-enter user session.
## Installation of microk8s addons
### addon dns
Enable the dns addon
```
$ microk8s enable dns
Enabling DNS
Applying manifest
serviceaccount/coredns created
configmap/coredns created
deployment.apps/coredns created
service/kube-dns created
clusterrole.rbac.authorization.k8s.io/coredns created
clusterrolebinding.rbac.authorization.k8s.io/coredns created
Restarting kubelet
[sudo] password for dgo: 
DNS is enabled
```
Verify the deployment, replicaset, pod and service
```
$ kubectl -n kube-system get events
$ kubectl -n kube-system get deployment
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
coredns   1/1     1            1           12m
$ kubectl -n kube-system get replicaset
NAME                 DESIRED   CURRENT   READY   AGE
coredns-588fd544bf   1         1         1       12m
$ kubectl -n kube-system get pods
NAME                       READY   STATUS    RESTARTS   AGE
coredns-588fd544bf-cstvl   1/1     Running   0          12m
$ kubectl -n kube-system get svc
NAME       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.152.183.10   <none>        53/UDP,53/TCP,9153/TCP   12m
```
### addon storage
Enable and verifiy storage addon
```
$ microk8s enable storage
Enabling default storage class
deployment.apps/hostpath-provisioner created
storageclass.storage.k8s.io/microk8s-hostpath created
serviceaccount/microk8s-hostpath created
clusterrole.rbac.authorization.k8s.io/microk8s-hostpath created
clusterrolebinding.rbac.authorization.k8s.io/microk8s-hostpath created
Storage will be available soon
$ kubectl -n kube-system get events
$ kubectl -n kube-system get deployment
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
coredns                1/1     1            1           14m
hostpath-provisioner   1/1     1            1           18s
$ kubectl -n kube-system get replicaset
NAME                              DESIRED   CURRENT   READY   AGE
coredns-588fd544bf                1         1         1       14m
hostpath-provisioner-75fdc8fccd   1         1         1       26s
$ kubectl -n kube-system get pods
NAME                                    READY   STATUS    RESTARTS   AGE
coredns-588fd544bf-cstvl                1/1     Running   0          14m
hostpath-provisioner-75fdc8fccd-nnzrl   1/1     Running   0          33s
$ kubectl -n kube-system get svc
NAME       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.152.183.10   <none>        53/UDP,53/TCP,9153/TCP   14m
$ kubectl -n kube-system get storageclass
NAME                          PROVISIONER            RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
microk8s-hostpath (default)   microk8s.io/hostpath   Delete          Immediate           false                  46s
```
### addon ingress
Enable and verify ingress addon
```
$ microk8s enable ingress
Enabling Ingress
namespace/ingress created
serviceaccount/nginx-ingress-microk8s-serviceaccount created
clusterrole.rbac.authorization.k8s.io/nginx-ingress-microk8s-clusterrole created
role.rbac.authorization.k8s.io/nginx-ingress-microk8s-role created
clusterrolebinding.rbac.authorization.k8s.io/nginx-ingress-microk8s created
rolebinding.rbac.authorization.k8s.io/nginx-ingress-microk8s created
configmap/nginx-load-balancer-microk8s-conf created
daemonset.apps/nginx-ingress-microk8s-controller created
Ingress is enabled
$ kubectl -n ingress get daemonset
NAME                                DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
nginx-ingress-microk8s-controller   1         1         1       1            1           <none>          76s
$ kubectl -n ingress describe daemonsets.apps nginx-ingress-microk8s-controller 
Name:           nginx-ingress-microk8s-controller
Selector:       name=nginx-ingress-microk8s
Node-Selector:  <none>
Labels:         microk8s-application=nginx-ingress-microk8s
Annotations:    deprecated.daemonset.template.generation: 1
Desired Number of Nodes Scheduled: 1
Current Number of Nodes Scheduled: 1
Number of Nodes Scheduled with Up-to-date Pods: 1
Number of Nodes Scheduled with Available Pods: 1
Number of Nodes Misscheduled: 0
Pods Status:  1 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:           name=nginx-ingress-microk8s
  Service Account:  nginx-ingress-microk8s-serviceaccount
  Containers:
   nginx-ingress-microk8s:
    Image:       quay.io/kubernetes-ingress-controller/nginx-ingress-controller-amd64:0.25.1
    Ports:       80/TCP, 443/TCP
    Host Ports:  80/TCP, 443/TCP
    Args:
      /nginx-ingress-controller
      --configmap=$(POD_NAMESPACE)/nginx-load-balancer-microk8s-conf
      --publish-status-address=127.0.0.1
    Liveness:  http-get http://:10254/healthz delay=30s timeout=5s period=10s #success=1 #failure=3
    Environment:
      POD_NAME:        (v1:metadata.name)
      POD_NAMESPACE:   (v1:metadata.namespace)
    Mounts:           <none>
  Volumes:            <none>
Events:
  Type    Reason            Age   From                  Message
  ----    ------            ----  ----                  -------
  Normal  SuccessfulCreate  95s   daemonset-controller  Created pod: nginx-ingress-microk8s-controller-98zl5
$ kubectl -n ingress get pods -o wide
NAME                                      READY   STATUS    RESTARTS   AGE   IP              NODE             NOMINATED NODE   READINESS GATES
nginx-ingress-microk8s-controller-98zl5   1/1     Running   0          12m   192.168.0.133   dgo-virtualbox   <none>           <none>
```
```
$ curl -v http://localhost
*   Trying 127.0.0.1:80...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 80 (#0)
> GET / HTTP/1.1
> Host: localhost
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 404 Not Found
< Server: openresty/1.15.8.1
< Date: Mon, 11 May 2020 16:19:35 GMT
< Content-Type: text/html
< Content-Length: 159
< Connection: keep-alive
< 
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>openresty/1.15.8.1</center>
</body>
</html>
* Connection #0 to host localhost left intact
```
```
$ curl -kv https://localhost
*   Trying 127.0.0.1:443...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/certs/ca-certificates.crt
  CApath: /etc/ssl/certs
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES256-GCM-SHA384
* ALPN, server accepted to use h2
* Server certificate:
*  subject: O=Acme Co; CN=Kubernetes Ingress Controller Fake Certificate
*  start date: May 11 16:05:16 2020 GMT
*  expire date: May 11 16:05:16 2021 GMT
*  issuer: O=Acme Co; CN=Kubernetes Ingress Controller Fake Certificate
*  SSL certificate verify result: unable to get local issuer certificate (20), continuing anyway.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x563139dbbdb0)
> GET / HTTP/2
> Host: localhost
> user-agent: curl/7.68.0
> accept: */*
> 
* Connection state changed (MAX_CONCURRENT_STREAMS == 128)!
< HTTP/2 404 
< server: openresty/1.15.8.1
< date: Mon, 11 May 2020 16:19:49 GMT
< content-type: text/html
< content-length: 159
< strict-transport-security: max-age=15724800; includeSubDomains
< 
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>openresty/1.15.8.1</center>
</body>
</html>
* Connection #0 to host localhost left intact
```

## App Deployments
[nginx webserver example](test-my-webserver.md)
