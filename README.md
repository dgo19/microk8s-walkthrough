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
## App Deployments
### nginx webserver
Create namespace and deployment for nginx
``` 
$ kubectl create namespace test
namespace/test created
$ kubectl get namespace
NAME              STATUS   AGE
default           Active   40m
kube-node-lease   Active   40m
kube-public       Active   40m
kube-system       Active   40m
test              Active   6s
$ kubectl -n test create deployment my-webserver --image=nginx
deployment.apps/my-webserver created
$ kubectl -n test get all
NAME                                READY   STATUS    RESTARTS   AGE
pod/my-webserver-6859dc4665-b52b8   1/1     Running   0          27s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-webserver   1/1     1            1           27s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/my-webserver-6859dc4665   1         1         1       27s
``` 
Access the new pod
``` 
$ ping 10.1.20.4
PING 10.1.20.4 (10.1.20.4) 56(84) bytes of data.
64 bytes from 10.1.20.4: icmp_seq=1 ttl=64 time=0.111 ms
64 bytes from 10.1.20.4: icmp_seq=2 ttl=64 time=0.086 ms
64 bytes from 10.1.20.4: icmp_seq=3 ttl=64 time=0.029 ms
^C
--- 10.1.20.4 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2046ms
rtt min/avg/max/mdev = 0.029/0.075/0.111/0.034 ms
$ curl -vvv http://10.1.20.4
*   Trying 10.1.20.4:80...
* TCP_NODELAY set
* Connected to 10.1.20.4 (10.1.20.4) port 80 (#0)
> GET / HTTP/1.1
> Host: 10.1.20.4
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: nginx/1.17.10
< Date: Mon, 11 May 2020 14:52:29 GMT
< Content-Type: text/html
< Content-Length: 612
< Last-Modified: Tue, 14 Apr 2020 14:19:26 GMT
< Connection: keep-alive
< ETag: "5e95c66e-264"
< Accept-Ranges: bytes
< 
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
* Connection #0 to host 10.1.20.4 left intact
``` 
Inspect the pod logs
``` 
$ kubectl -n test get pods
NAME                            READY   STATUS    RESTARTS   AGE
my-webserver-6859dc4665-b52b8   1/1     Running   0          21m
dgo@dgo-VirtualBox:~/test$ kubectl -n test logs my-webserver-6859dc4665-b52b8 
10.1.20.1 - - [11/May/2020:14:52:29 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
10.1.20.1 - - [11/May/2020:14:54:38 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:76.0) Gecko/20100101 Firefox/76.0" "-"
10.1.20.1 - - [11/May/2020:14:54:38 +0000] "GET /favicon.ico HTTP/1.1" 404 154 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:76.0) Gecko/20100101 Firefox/76.0" "-"
2020/05/11 14:54:38 [error] 6#6: *2 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 10.1.20.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "10.1.20.4"
``` 
Delete the pod - it will be restarted by kubernetes
``` 
$ kubectl -n test get pods
NAME                            READY   STATUS    RESTARTS   AGE
my-webserver-6859dc4665-b52b8   1/1     Running   0          22m
$ kubectl -n test delete pod my-webserver-6859dc4665-b52b8 
pod "my-webserver-6859dc4665-b52b8" deleted
$ kubectl -n test get pods
NAME                            READY   STATUS    RESTARTS   AGE
my-webserver-6859dc4665-mdqkg   1/1     Running   0          3s
$ kubectl -n test logs my-webserver-6859dc4665-mdqkg 
$ kubectl -n test get events
LAST SEEN   TYPE     REASON              OBJECT                               MESSAGE
23m         Normal   Scheduled           pod/my-webserver-6859dc4665-b52b8    Successfully assigned test/my-webserver-6859dc4665-b52b8 to dgo-virtualbox
23m         Normal   Pulling             pod/my-webserver-6859dc4665-b52b8    Pulling image "nginx"
22m         Normal   Pulled              pod/my-webserver-6859dc4665-b52b8    Successfully pulled image "nginx"
22m         Normal   Created             pod/my-webserver-6859dc4665-b52b8    Created container nginx
22m         Normal   Started             pod/my-webserver-6859dc4665-b52b8    Started container nginx
25s         Normal   Killing             pod/my-webserver-6859dc4665-b52b8    Stopping container nginx
25s         Normal   Scheduled           pod/my-webserver-6859dc4665-mdqkg    Successfully assigned test/my-webserver-6859dc4665-mdqkg to dgo-virtualbox
25s         Normal   Pulling             pod/my-webserver-6859dc4665-mdqkg    Pulling image "nginx"
24s         Normal   Pulled              pod/my-webserver-6859dc4665-mdqkg    Successfully pulled image "nginx"
24s         Normal   Created             pod/my-webserver-6859dc4665-mdqkg    Created container nginx
23s         Normal   Started             pod/my-webserver-6859dc4665-mdqkg    Started container nginx
23m         Normal   SuccessfulCreate    replicaset/my-webserver-6859dc4665   Created pod: my-webserver-6859dc4665-b52b8
25s         Normal   SuccessfulCreate    replicaset/my-webserver-6859dc4665   Created pod: my-webserver-6859dc4665-mdqkg
23m         Normal   ScalingReplicaSet   deployment/my-webserver              Scaled up replica set my-webserver-6859dc4665 to 1
``` 
Scale the pod and increase the replicas to 2.
``` 
$ kubectl -n test scale deployment --replicas=2 my-webserver
deployment.apps/my-webserver scaled
$ kubectl -n test get deployment
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
my-webserver   2/2     2            2           27m
$ kubectl -n test get replicaset
NAME                      DESIRED   CURRENT   READY   AGE
my-webserver-6859dc4665   2         2         2       27m
$ kubectl -n test get pods -o wide
NAME                            READY   STATUS    RESTARTS   AGE     IP          NODE             NOMINATED NODE   READINESS GATES
my-webserver-6859dc4665-mdqkg   1/1     Running   0          4m33s   10.1.20.5   dgo-virtualbox   <none>           <none>
my-webserver-6859dc4665-xmbsh   1/1     Running   0          20s     10.1.20.6   dgo-virtualbox   <none>           <none>
$ curl http://10.1.20.5 | head -5
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   612  100   612    0     0   597k      0 --:--:-- --:--:-- --:--:--  597k
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
$ curl http://10.1.20.6 | head -5
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   612  100   612    0     0   597k      0 --:--:-- --:--:-- --:--:--  597k
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
``` 
