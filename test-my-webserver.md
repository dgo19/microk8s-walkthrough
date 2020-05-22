# microk8s walkthrough - my-webserver

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
$ kubectl -n test get pods -o wide
NAME                            READY   STATUS    RESTARTS   AGE     IP          NODE             NOMINATED NODE   READINESS GATES
my-webserver-6859dc4665-b52b8   1/1     Running   0          3m33s   10.1.20.4   dgo-virtualbox   <none>           <none>
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
Create service for my-webserver.
``` 
$ kubectl -n test create service clusterip my-webserver --tcp=80:80
service/my-webserver created
$ kubectl -n test get svc
NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
my-webserver   ClusterIP   10.152.183.183   <none>        80/TCP    42s
$ kubectl -n test describe svc my-webserver 
Name:              my-webserver
Namespace:         test
Labels:            app=my-webserver
Annotations:       <none>
Selector:          app=my-webserver
Type:              ClusterIP
IP:                10.152.183.183
Port:              80-80  80/TCP
TargetPort:        80/TCP
Endpoints:         10.1.20.5:80,10.1.20.6:80
Session Affinity:  None
Events:            <none>
$ curl http://10.152.183.183 | head -5
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   612  100   612    0     0   597k      0 --:--:-- --:--:-- --:--:--  597k
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
$ curl http://10.152.183.183 | head -5
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   612  100   612    0     0   597k      0 --:--:-- --:--:-- --:--:--  597k
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
$ curl http://10.152.183.183 | head -5
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   612  100   612    0     0   597k      0 --:--:-- --:--:-- --:--:--  597k
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
$ curl http://10.152.183.183 | head -5
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   612  100   612    0     0   597k      0 --:--:-- --:--:-- --:--:--  597k
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
$ kubectl -n test logs my-webserver-6859dc4665-
my-webserver-6859dc4665-mdqkg  my-webserver-6859dc4665-xmbsh  
$ kubectl -n test logs my-webserver-6859dc4665-mdqkg 
10.1.20.1 - - [11/May/2020:15:14:57 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
10.1.20.1 - - [11/May/2020:15:15:07 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
10.1.20.1 - - [11/May/2020:15:15:14 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
10.1.20.1 - - [11/May/2020:15:23:31 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
10.1.20.1 - - [11/May/2020:15:23:33 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
$ kubectl -n test logs my-webserver-6859dc4665-xmbsh 
10.1.20.1 - - [11/May/2020:15:15:20 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
10.1.20.1 - - [11/May/2020:15:23:32 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
10.1.20.1 - - [11/May/2020:15:23:33 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
``` 
Create and test ingress.
``` 
$ cat <<EOF | kubectl -n test apply -f -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-my-webserver
spec:
  rules:
  - host: my-webserver.microk8s.local
    http:
      paths:
      - backend:
          serviceName: my-webserver
          servicePort: 80
EOF
ingress.extensions/ingress-my-webserver created
``` 
``` 
$ curl -v --header "Host: my-webserver.microk8s.local" http://localhost
*   Trying 127.0.0.1:80...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 80 (#0)
> GET / HTTP/1.1
> Host: my-webserver.microk8s.local
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: openresty/1.15.8.1
< Date: Mon, 11 May 2020 16:28:28 GMT
< Content-Type: text/html
< Content-Length: 612
< Connection: keep-alive
< Vary: Accept-Encoding
< Last-Modified: Tue, 14 Apr 2020 14:19:26 GMT
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
* Connection #0 to host localhost left intact
``` 
``` 
$ curl -kv --header "Host: my-webserver.microk8s.local" https://localhost
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
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* ALPN, server accepted to use h2
* Server certificate:
*  subject: O=nginx ingress; OU=nginx ingress; CN=*.microk8s.local
*  start date: May 13 14:26:15 2020 GMT
*  expire date: May 11 14:26:15 2030 GMT
*  issuer: O=nginx ingress; OU=nginx ingress; CN=*.microk8s.local
*  SSL certificate verify result: self signed certificate (18), continuing anyway.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x559f5eaf7db0)
> GET / HTTP/2
> Host: my-webserver.microk8s.local
> user-agent: curl/7.68.0
> accept: */*
> 
* Connection state changed (MAX_CONCURRENT_STREAMS == 128)!
< HTTP/2 200 
< server: nginx/1.17.10
< date: Wed, 13 May 2020 14:42:28 GMT
< content-type: text/html
< content-length: 612
< vary: Accept-Encoding
< last-modified: Tue, 14 Apr 2020 14:19:26 GMT
< etag: "5e95c66e-264"
< accept-ranges: bytes
< strict-transport-security: max-age=15724800; includeSubDomains
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
* Connection #0 to host localhost left intact
``` 
Delete the test namespace
``` 
$ kubectl delete namespace test
namespace "test" deleted
``` 
Create my-webserver by using kustomize build. Examples at applications/test-my-webserver
``` 
$ cd microk8s-walkthrough/applications/test-my-webserver/
$ kustomize build . | kubectl apply -f -
namespace/test created
service/my-webserver created
deployment.apps/my-webserver created
ingress.extensions/ingress-my-webserver created
$ kubectl -n test get pods
NAME                            READY   STATUS    RESTARTS   AGE
my-webserver-6859dc4665-4hwpq   1/1     Running   0          9s
my-webserver-6859dc4665-mcspx   1/1     Running   0          9s
``` 
Create my-webserver by kubectl -k. Examples at applications/test-my-webserver
``` 
$ kubectl delete namespace test
namespace "test" deleted
$ kubectl apply -k .
namespace/test created
service/my-webserver created
deployment.apps/my-webserver created
ingress.extensions/ingress-my-webserver created
$ kubectl -n test get pods
NAME                            READY   STATUS    RESTARTS   AGE
my-webserver-6859dc4665-pglw2   1/1     Running   0          56s
my-webserver-6859dc4665-qcr4k   1/1     Running   0          56s
``` 
