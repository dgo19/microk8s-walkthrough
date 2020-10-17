## Installation of microk8s
```
$ sudo snap install microk8s --classic
microk8s (1.19/stable) v1.19.2 from Canonical installed
```
Add user to microk8s group and set alias for kubectl
```
$ sudo usermod -a -G microk8s dgo
$ echo "alias kubectl='microk8s kubectl'" >> ~/.bashrc
$ echo 'source <(microk8s kubectl completion bash)' >> ~/.bashrc
```
Get kustomize
```
$ mkdir ~/bin
$ curl -L https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize%2Fv3.8.5/kustomize_v3.8.5_linux_amd64.tar.gz | tar xvzC ~/bin -f -
```
Re-enter user session.

## Wildcard DNS for ingress
Add wildcard DNS entry for ingress:
The wildcard domain *.microk8s.local will be used in this walkthrough for ingress. Please add a the wilcard domain to your DNS server pointing to the IP of the microk8s node.

Test the wildcard domain
```
$ ping -c1 hello-wildcard.microk8s.local
PING hello-wildcard.microk8s.local (192.168.0.133) 56(84) bytes of data.
64 bytes from dgo-VirtualBox (192.168.0.133): icmp_seq=1 ttl=64 time=0.023 ms

--- hello-wildcard.microk8s.local ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.023/0.023/0.023/0.000 ms

$ ping -c1 hello-wildcard2.microk8s.local
PING hello-wildcard2.microk8s.local (192.168.0.133) 56(84) bytes of data.
64 bytes from dgo-VirtualBox (192.168.0.133): icmp_seq=1 ttl=64 time=0.023 ms

--- hello-wildcard2.microk8s.local ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.023/0.023/0.023/0.000 ms
```
In case of a single ubuntu VM, you add the required subdomains to /etc/hosts
```
sudo sed -i '/^127.0.0.1/ s/$/ my-webserver\.microk8s\.local/' /etc/hosts
```
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
DNS is enabled
```
Verify the deployment, replicaset, pod and service
```
$ kubectl -n kube-system get events
$ kubectl -n kube-system get deployment
NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
calico-kube-controllers   1/1     1            1           3m37s
coredns                   1/1     1            1           52s
$ kubectl -n kube-system get replicaset
NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
calico-kube-controllers   1/1     1            1           3m37s
coredns                   1/1     1            1           52s
$ kubectl -n kube-system get pods
NAME                                      READY   STATUS    RESTARTS   AGE
calico-node-227wh                         1/1     Running   1          4m23s
calico-kube-controllers-847c8c99d-9nxfd   1/1     Running   0          4m24s
coredns-86f78bb79c-xrf9h                  1/1     Running   0          104s
$ kubectl -n kube-system get svc
NAME       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.152.183.10   <none>        53/UDP,53/TCP,9153/TCP   2m8s
```
### addon storage
Enable and verify storage addon
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
NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
calico-kube-controllers   1/1     1            1           6m2s
coredns                   1/1     1            1           3m17s
hostpath-provisioner      1/1     1            1           13s
$ kubectl -n kube-system get replicaset
NAME                                DESIRED   CURRENT   READY   AGE
calico-kube-controllers-847c8c99d   1         1         1       6m18s
coredns-86f78bb79c                  1         1         1       3m38s
hostpath-provisioner-5c65fbdb4f     1         1         1       34s
$ kubectl -n kube-system get pods
NAME                                DESIRED   CURRENT   READY   AGE
calico-kube-controllers-847c8c99d   1         1         1       6m18s
coredns-86f78bb79c                  1         1         1       3m38s
hostpath-provisioner-5c65fbdb4f     1         1         1       34s
$ kubectl -n kube-system get svc
NAME       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.152.183.10   <none>        53/UDP,53/TCP,9153/TCP   4m16s
$ kubectl -n kube-system get storageclass
NAME                          PROVISIONER            RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
microk8s-hostpath (default)   microk8s.io/hostpath   Delete          Immediate           false                  89s
```
### addon ingress
Enable and verify ingress addon
```
$ git clone https://github.com/dgo19/microk8s-walkthrough.git
$ cd microk8s-walkthrough/applications/ingress-nginx
$ kubectl apply -k .
namespace/ingress-nginx created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
serviceaccount/ingress-nginx-admission created
serviceaccount/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
configmap/ingress-nginx-controller created
secret/tls-secret created
service/ingress-nginx-controller-admission created
service/ingress-nginx-controller created
deployment.apps/ingress-nginx-controller created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created

$ kubectl -n ingress-nginx get pods
NAME                                        READY   STATUS              RESTARTS   AGE
ingress-nginx-controller-76b4d74777-lbgwq   0/1     ContainerCreating   0          28s
ingress-nginx-admission-create-fvw79        0/1     Completed           0          28s
ingress-nginx-admission-patch-9nwln         0/1     Completed           0          28s

$ kubectl -n ingress-nginx get pods
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-fvw79        0/1     Completed   0          58s
ingress-nginx-admission-patch-9nwln         0/1     Completed   0          58s
ingress-nginx-controller-76b4d74777-lbgwq   1/1     Running     0          58s

$ kubectl -n ingress-nginx get deployment
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
ingress-nginx-controller   1/1     1            1           112s

$ kubectl -n ingress-nginx get replicaset
NAME                                  DESIRED   CURRENT   READY   AGE
ingress-nginx-controller-76b4d74777   1         1         1       2m

$ kubectl -n ingress-nginx get pods -o wide
NAME                                        READY   STATUS      RESTARTS   AGE    IP              NODE             NOMINATED NODE   READINESS GATES
ingress-nginx-admission-create-fvw79        0/1     Completed   0          2m6s   10.1.100.197    dgo-virtualbox   <none>           <none>
ingress-nginx-admission-patch-9nwln         0/1     Completed   0          2m6s   10.1.100.198    dgo-virtualbox   <none>           <none>
ingress-nginx-controller-76b4d74777-lbgwq   1/1     Running     0          2m6s   192.168.0.133   dgo-virtualbox   <none>           <none>
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
< Date: Sat, 17 Oct 2020 11:38:45 GMT
< Content-Type: text/html
< Content-Length: 146
< Connection: keep-alive
< 
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
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
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* ALPN, server accepted to use h2
* Server certificate:
*  subject: O=nginx ingress; OU=nginx ingress; CN=*.microk8s.local
*  start date: May 13 14:26:15 2020 GMT
*  expire date: May 11 14:26:15 2030 GMT
*  issuer: O=nginx ingress; OU=nginx ingress; CN=*.microk8s.local
*  SSL certificate verify ok.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x55932d181db0)
> GET / HTTP/2
> Host: localhost
> user-agent: curl/7.68.0
> accept: */*
> 
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* old SSL session ID is stale, removing
* Connection state changed (MAX_CONCURRENT_STREAMS == 128)!
< HTTP/2 404 
< date: Sat, 17 Oct 2020 11:39:22 GMT
< content-type: text/html
< content-length: 146
< strict-transport-security: max-age=15724800; includeSubDomains
< 
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>
* Connection #0 to host localhost left intact
```
