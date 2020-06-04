# Argo CD Deployment & first steps
Deploy Argo CD.
```
$ cd microk8s-walkthrough/argocd/
$ kubectl apply -k .
namespace/argocd created
customresourcedefinition.apiextensions.k8s.io/applications.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/appprojects.argoproj.io created
serviceaccount/argocd-application-controller created
serviceaccount/argocd-dex-server created
serviceaccount/argocd-server created
role.rbac.authorization.k8s.io/argocd-application-controller created
role.rbac.authorization.k8s.io/argocd-dex-server created
role.rbac.authorization.k8s.io/argocd-server created
clusterrole.rbac.authorization.k8s.io/argocd-application-controller created
clusterrole.rbac.authorization.k8s.io/argocd-server created
rolebinding.rbac.authorization.k8s.io/argocd-application-controller created
rolebinding.rbac.authorization.k8s.io/argocd-dex-server created
rolebinding.rbac.authorization.k8s.io/argocd-server created
clusterrolebinding.rbac.authorization.k8s.io/argocd-application-controller created
clusterrolebinding.rbac.authorization.k8s.io/argocd-server created
configmap/argocd-cm created
configmap/argocd-rbac-cm created
configmap/argocd-ssh-known-hosts-cm created
configmap/argocd-tls-certs-cm created
secret/argocd-secret created
service/argocd-dex-server created
service/argocd-metrics created
service/argocd-redis created
service/argocd-repo-server created
service/argocd-server-metrics created
service/argocd-server created
deployment.apps/argocd-application-controller created
deployment.apps/argocd-dex-server created
deployment.apps/argocd-redis created
deployment.apps/argocd-repo-server created
deployment.apps/argocd-server created
ingress.extensions/ingress-argocd-server created
```
verify the pods are running and review the deployment
```
$ kubectl -n argocd get pods
NAME                                             READY   STATUS              RESTARTS   AGE
argocd-application-controller-5b47f689bd-w5bkm   0/1     ContainerCreating   0          64s
argocd-dex-server-65dfb768d6-w2x69               0/1     Init:0/1            0          64s
argocd-redis-6d7f9df848-k77ql                    0/1     ContainerCreating   0          64s
argocd-repo-server-56fdd6cf6d-jws59              0/1     ContainerCreating   0          64s
argocd-server-6ddc6bcc8d-4lgr2                   0/1     ContainerCreating   0          64s
$ kubectl -n argocd get pods
NAME                                             READY   STATUS    RESTARTS   AGE
argocd-application-controller-5b47f689bd-w5bkm   1/1     Running   0          3m6s
argocd-dex-server-65dfb768d6-w2x69               1/1     Running   0          3m6s
argocd-redis-6d7f9df848-k77ql                    1/1     Running   0          3m6s
argocd-repo-server-56fdd6cf6d-jws59              1/1     Running   0          3m6s
argocd-server-6ddc6bcc8d-4lgr2                   1/1     Running   0          3m6s
$ kubectl -n argocd get svc
NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
argocd-dex-server       ClusterIP   10.152.183.152   <none>        5556/TCP,5557/TCP,5558/TCP   3m32s
argocd-metrics          ClusterIP   10.152.183.81    <none>        8082/TCP                     3m32s
argocd-redis            ClusterIP   10.152.183.101   <none>        6379/TCP                     3m32s
argocd-repo-server      ClusterIP   10.152.183.30    <none>        8081/TCP,8084/TCP            3m32s
argocd-server           ClusterIP   10.152.183.16    <none>        80/TCP,443/TCP               3m32s
argocd-server-metrics   ClusterIP   10.152.183.133   <none>        8083/TCP                     3m32s
$ kubectl -n argocd get ingress
NAME                    CLASS    HOSTS                   ADDRESS         PORTS   AGE
ingress-argocd-server   <none>   argocd.microk8s.local   192.168.0.133   80      3m36s
```
Download the commandline client
```
$ curl -L -o ~/bin/argocd https://github.com/argoproj/argo-cd/releases/download/v1.5.4/argocd-linux-amd64
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   631  100   631    0     0   6572      0 --:--:-- --:--:-- --:--:--  6572
100 65.1M  100 65.1M    0     0  9315k      0  0:00:07  0:00:07 --:--:-- 11.4M
$ chmod +x ~/bin/argocd
```
Open the web-ui (https://argocd.microk8s.local/) or login by command line client.
The following tasks can be done by cli or web ui.
```
$ argocd login --insecure --username admin --password admin argocd.microk8s.local
'admin' logged in successfully
Context 'argocd.microk8s.local' updated
```
Review the cluster, project, repo and apps.
```

$ argocd cluster list
SERVER                          NAME  VERSION  STATUS      MESSAGE
https://kubernetes.default.svc                 Successful  
```
```
$ argocd repo list
TYPE  NAME                        REPO                                               INSECURE  LFS    CREDS  STATUS      MESSAGE
git   dgo19-microk8s-walkthrough  https://github.com/dgo19/microk8s-walkthrough.git  false     false  false  Successful  
```
```
$ argocd proj list
NAME     DESCRIPTION  DESTINATIONS  SOURCES  CLUSTER-RESOURCE-WHITELIST  NAMESPACE-RESOURCE-BLACKLIST  ORPHANED-RESOURCES
default               *,*           *        */*                         <none>                        disabled
```
```
$ argocd app list
NAME  CLUSTER  NAMESPACE  PROJECT  STATUS  HEALTH  SYNCPOLICY  CONDITIONS  REPO  PATH  TARGET
```
Create the first argocd application by applying the yaml-file - argocd itself.
```
$ kubectl apply -f application-argocd.yaml 
application.argoproj.io/argocd created
```
Now there is an application argocd in status OutOfSync.
```
$ argocd app list
NAME    CLUSTER                         NAMESPACE  PROJECT  STATUS     HEALTH   SYNCPOLICY  CONDITIONS  REPO                                               PATH    TARGET
argocd  https://kubernetes.default.svc  argocd     default  OutOfSync  Healthy  <none>      <none>      https://github.com/dgo19/microk8s-walkthrough.git  argocd  HEAD
```
```
$ argocd app diff argocd
===== /ConfigMap argocd/argocd-tls-certs-cm ======
4a5
>     app.kubernetes.io/instance: argocd
===== /Service argocd/argocd-server ======
5a6
>     app.kubernetes.io/instance: argocd
===== /Service argocd/argocd-server-metrics ======
5a6
>     app.kubernetes.io/instance: argocd
===== apps/Deployment argocd/argocd-application-controller ======
5a6
>     app.kubernetes.io/instance: argocd
===== rbac.authorization.k8s.io/ClusterRole /argocd-server ======
5a6
>     app.kubernetes.io/instance: argocd
===== /ConfigMap argocd/argocd-ssh-known-hosts-cm ======
13a14
>     app.kubernetes.io/instance: argocd
===== /ServiceAccount argocd/argocd-application-controller ======
5a6
>     app.kubernetes.io/instance: argocd
===== rbac.authorization.k8s.io/RoleBinding argocd/argocd-dex-server ======
5a6
>     app.kubernetes.io/instance: argocd
===== rbac.authorization.k8s.io/RoleBinding argocd/argocd-server ======
5a6
>     app.kubernetes.io/instance: argocd
===== /Service argocd/argocd-redis ======
5a6
>     app.kubernetes.io/instance: argocd
===== /Namespace /argocd ======
3a4,5
>   labels:
>     app.kubernetes.io/instance: argocd
===== rbac.authorization.k8s.io/Role argocd/argocd-dex-server ======
5a6
>     app.kubernetes.io/instance: argocd
===== /Secret argocd/argocd-secret ======
7a8
>     app.kubernetes.io/instance: argocd
===== apps/Deployment argocd/argocd-dex-server ======
5a6
>     app.kubernetes.io/instance: argocd
===== /ServiceAccount argocd/argocd-server ======
5a6
>     app.kubernetes.io/instance: argocd
===== rbac.authorization.k8s.io/ClusterRole /argocd-application-controller ======
5a6
>     app.kubernetes.io/instance: argocd
===== rbac.authorization.k8s.io/ClusterRoleBinding /argocd-server ======
5a6
>     app.kubernetes.io/instance: argocd
===== /ConfigMap argocd/argocd-rbac-cm ======
4a5
>     app.kubernetes.io/instance: argocd
===== rbac.authorization.k8s.io/Role argocd/argocd-application-controller ======
5a6
>     app.kubernetes.io/instance: argocd
===== rbac.authorization.k8s.io/RoleBinding argocd/argocd-application-controller ======
5a6
>     app.kubernetes.io/instance: argocd
===== /ConfigMap argocd/argocd-cm ======
9a10
>     app.kubernetes.io/instance: argocd
===== apps/Deployment argocd/argocd-redis ======
5a6
>     app.kubernetes.io/instance: argocd
===== apps/Deployment argocd/argocd-server ======
5a6
>     app.kubernetes.io/instance: argocd
===== extensions/Ingress argocd/ingress-argocd-server ======
8a9,10
>   labels:
>     app.kubernetes.io/instance: argocd
===== rbac.authorization.k8s.io/Role argocd/argocd-server ======
5a6
>     app.kubernetes.io/instance: argocd
===== rbac.authorization.k8s.io/ClusterRoleBinding /argocd-application-controller ======
5a6
>     app.kubernetes.io/instance: argocd
===== /Service argocd/argocd-repo-server ======
5a6
>     app.kubernetes.io/instance: argocd
===== /ServiceAccount argocd/argocd-dex-server ======
5a6
>     app.kubernetes.io/instance: argocd
===== /Service argocd/argocd-metrics ======
5a6
>     app.kubernetes.io/instance: argocd
===== apps/Deployment argocd/argocd-repo-server ======
5a6
>     app.kubernetes.io/instance: argocd
===== /Service argocd/argocd-dex-server ======
5a6
>     app.kubernetes.io/instance: argocd
```
The argocd labels are missing in the deployed application and this is fixed by an argocd sync.
```
$ argocd app sync argocd
TIMESTAMP                  GROUP                            KIND                 NAMESPACE                  NAME             STATUS    HEALTH        HOOK  MESSAGE
2020-06-04T17:17:58+02:00  rbac.authorization.k8s.io  RoleBinding                   argocd     argocd-dex-server           OutOfSync                       
2020-06-04T17:17:58+02:00                              ConfigMap                    argocd  argocd-ssh-known-hosts-cm      OutOfSync                       
2020-06-04T17:17:58+02:00                              Namespace                                          argocd           OutOfSync                       
2020-06-04T17:17:58+02:00                                Service                    argocd     argocd-dex-server           OutOfSync  Healthy              
2020-06-04T17:17:58+02:00                                Service                    argocd        argocd-metrics           OutOfSync  Healthy              
2020-06-04T17:17:58+02:00                                Service                    argocd          argocd-redis           OutOfSync  Healthy              
2020-06-04T17:17:58+02:00                             ServiceAccount                argocd  argocd-application-controller  OutOfSync                       
2020-06-04T17:17:58+02:00                              ConfigMap                    argocd        argocd-rbac-cm           OutOfSync                       
2020-06-04T17:17:58+02:00   apps                      Deployment                    argocd         argocd-server           OutOfSync  Healthy              
2020-06-04T17:17:58+02:00                             ServiceAccount                argocd         argocd-server           OutOfSync                       
2020-06-04T17:17:58+02:00   apps                      Deployment                    argocd  argocd-application-controller  OutOfSync  Healthy              
2020-06-04T17:17:58+02:00   apps                      Deployment                    argocd          argocd-redis           OutOfSync  Healthy              
2020-06-04T17:17:58+02:00  rbac.authorization.k8s.io  ClusterRoleBinding                    argocd-application-controller  OutOfSync                       
2020-06-04T17:17:58+02:00  rbac.authorization.k8s.io        Role                    argocd         argocd-server           OutOfSync                       
2020-06-04T17:17:58+02:00                             ServiceAccount                argocd     argocd-dex-server           OutOfSync                       
2020-06-04T17:17:58+02:00  rbac.authorization.k8s.io        Role                    argocd     argocd-dex-server           OutOfSync                       
2020-06-04T17:17:58+02:00                                Service                    argocd         argocd-server           OutOfSync  Healthy              
2020-06-04T17:17:58+02:00  apiextensions.k8s.io       CustomResourceDefinition              appprojects.argoproj.io          Synced                        
2020-06-04T17:17:58+02:00   apps                      Deployment                    argocd     argocd-dex-server           OutOfSync  Healthy              
2020-06-04T17:17:58+02:00   apps                      Deployment                    argocd    argocd-repo-server           OutOfSync  Healthy              
2020-06-04T17:17:58+02:00  extensions                    Ingress                    argocd  ingress-argocd-server          OutOfSync  Healthy              
2020-06-04T17:17:58+02:00  rbac.authorization.k8s.io  ClusterRole                                  argocd-server           OutOfSync                       
2020-06-04T17:17:58+02:00                                 Secret                    argocd         argocd-secret           OutOfSync                       
2020-06-04T17:17:58+02:00                                Service                    argocd    argocd-repo-server           OutOfSync  Healthy              
2020-06-04T17:17:58+02:00                                Service                    argocd  argocd-server-metrics          OutOfSync  Healthy              
2020-06-04T17:17:58+02:00  rbac.authorization.k8s.io  RoleBinding                   argocd  argocd-application-controller  OutOfSync                       
2020-06-04T17:17:58+02:00  rbac.authorization.k8s.io  RoleBinding                   argocd         argocd-server           OutOfSync                       
2020-06-04T17:17:58+02:00                              ConfigMap                    argocd             argocd-cm           OutOfSync                       
2020-06-04T17:17:58+02:00                              ConfigMap                    argocd   argocd-tls-certs-cm           OutOfSync                       
2020-06-04T17:17:58+02:00  apiextensions.k8s.io       CustomResourceDefinition              applications.argoproj.io         Synced                        
2020-06-04T17:17:58+02:00  rbac.authorization.k8s.io  ClusterRole                           argocd-application-controller  OutOfSync                       
2020-06-04T17:17:58+02:00  rbac.authorization.k8s.io  ClusterRoleBinding                           argocd-server           OutOfSync                       
2020-06-04T17:17:58+02:00  rbac.authorization.k8s.io        Role                    argocd  argocd-application-controller  OutOfSync                       
2020-06-04T17:18:00+02:00          Namespace                            argocd    Synced                       
2020-06-04T17:18:00+02:00          ConfigMap      argocd             argocd-cm         Synced                       
2020-06-04T17:18:00+02:00          ConfigMap      argocd  argocd-ssh-known-hosts-cm    Synced                       
2020-06-04T17:18:00+02:00             Secret      argocd         argocd-secret         Synced                       
2020-06-04T17:18:00+02:00          ConfigMap      argocd   argocd-tls-certs-cm         Synced                       
2020-06-04T17:18:00+02:00          ConfigMap      argocd        argocd-rbac-cm         Synced                       
2020-06-04T17:18:00+02:00         ServiceAccount      argocd  argocd-application-controller    Synced                       
2020-06-04T17:18:00+02:00         ServiceAccount      argocd         argocd-server             Synced                       
2020-06-04T17:18:00+02:00         ServiceAccount      argocd     argocd-dex-server             Synced                       
2020-06-04T17:18:01+02:00  rbac.authorization.k8s.io  ClusterRole              argocd-application-controller    Synced                       
2020-06-04T17:18:01+02:00  rbac.authorization.k8s.io  ClusterRole                     argocd-server             Synced                       
2020-06-04T17:18:01+02:00  rbac.authorization.k8s.io  ClusterRoleBinding              argocd-application-controller    Synced                       
2020-06-04T17:18:01+02:00  rbac.authorization.k8s.io        Role              argocd     argocd-dex-server             Synced                       
2020-06-04T17:18:01+02:00  rbac.authorization.k8s.io        Role              argocd  argocd-application-controller    Synced                       
2020-06-04T17:18:01+02:00  rbac.authorization.k8s.io  ClusterRoleBinding                     argocd-server             Synced                       
2020-06-04T17:18:01+02:00  rbac.authorization.k8s.io  RoleBinding      argocd     argocd-dex-server             Synced                       
2020-06-04T17:18:01+02:00  rbac.authorization.k8s.io  RoleBinding      argocd  argocd-application-controller    Synced                       
2020-06-04T17:18:01+02:00  rbac.authorization.k8s.io        Role       argocd         argocd-server             Synced                       
2020-06-04T17:18:01+02:00  rbac.authorization.k8s.io  RoleBinding      argocd         argocd-server             Synced                       
2020-06-04T17:18:02+02:00                              Namespace                    argocd                argocd            Running    Synced              namespace/argocd configured
2020-06-04T17:18:02+02:00                              ConfigMap                    argocd        argocd-rbac-cm             Synced                        configmap/argocd-rbac-cm configured
2020-06-04T17:18:02+02:00  rbac.authorization.k8s.io        Role                    argocd  argocd-application-controller    Synced                        role.rbac.authorization.k8s.io/argocd-application-controller reconciled. reconciliation required update. role.rbac.authorization.k8s.io/argocd-application-controller configured
2020-06-04T17:18:02+02:00  rbac.authorization.k8s.io        Role                    argocd     argocd-dex-server             Synced                        role.rbac.authorization.k8s.io/argocd-dex-server reconciled. reconciliation required update. role.rbac.authorization.k8s.io/argocd-dex-server configured
2020-06-04T17:18:02+02:00  rbac.authorization.k8s.io        Role                    argocd         argocd-server             Synced                        role.rbac.authorization.k8s.io/argocd-server reconciled. reconciliation required update. role.rbac.authorization.k8s.io/argocd-server configured
2020-06-04T17:18:02+02:00  extensions                    Ingress                    argocd  ingress-argocd-server          OutOfSync  Healthy              ingress.extensions/ingress-argocd-server configured
2020-06-04T17:18:02+02:00                             ServiceAccount                argocd  argocd-application-controller    Synced                        serviceaccount/argocd-application-controller configured
2020-06-04T17:18:02+02:00  rbac.authorization.k8s.io  ClusterRole                   argocd  argocd-application-controller   Running    Synced              clusterrole.rbac.authorization.k8s.io/argocd-application-controller reconciled. reconciliation required update. clusterrole.rbac.authorization.k8s.io/argocd-application-controller configured
2020-06-04T17:18:02+02:00  rbac.authorization.k8s.io  RoleBinding                   argocd         argocd-server             Synced                        rolebinding.rbac.authorization.k8s.io/argocd-server reconciled. reconciliation required update. rolebinding.rbac.authorization.k8s.io/argocd-server configured
2020-06-04T17:18:02+02:00  rbac.authorization.k8s.io  RoleBinding                   argocd     argocd-dex-server             Synced                        rolebinding.rbac.authorization.k8s.io/argocd-dex-server reconciled. reconciliation required update. rolebinding.rbac.authorization.k8s.io/argocd-dex-server configured
2020-06-04T17:18:02+02:00                                Service                    argocd          argocd-redis           OutOfSync  Healthy              service/argocd-redis configured
2020-06-04T17:18:02+02:00                                Service                    argocd     argocd-dex-server           OutOfSync  Healthy              service/argocd-dex-server configured
2020-06-04T17:18:02+02:00                              ConfigMap                    argocd   argocd-tls-certs-cm             Synced                        configmap/argocd-tls-certs-cm configured
2020-06-04T17:18:02+02:00                              ConfigMap                    argocd  argocd-ssh-known-hosts-cm        Synced                        configmap/argocd-ssh-known-hosts-cm configured
2020-06-04T17:18:02+02:00                                 Secret                    argocd         argocd-secret             Synced                        secret/argocd-secret configured
2020-06-04T17:18:02+02:00                                Service                    argocd  argocd-server-metrics          OutOfSync  Healthy              service/argocd-server-metrics configured
2020-06-04T17:18:02+02:00  apiextensions.k8s.io       CustomResourceDefinition      argocd  applications.argoproj.io        Running    Synced              customresourcedefinition.apiextensions.k8s.io/applications.argoproj.io unchanged
2020-06-04T17:18:02+02:00  rbac.authorization.k8s.io  ClusterRole                   argocd         argocd-server            Running    Synced              clusterrole.rbac.authorization.k8s.io/argocd-server reconciled. reconciliation required update. clusterrole.rbac.authorization.k8s.io/argocd-server configured
2020-06-04T17:18:02+02:00   apps                      Deployment                    argocd          argocd-redis           OutOfSync  Healthy              deployment.apps/argocd-redis configured
2020-06-04T17:18:02+02:00   apps                      Deployment                    argocd     argocd-dex-server           OutOfSync  Healthy              deployment.apps/argocd-dex-server configured
2020-06-04T17:18:02+02:00                             ServiceAccount                argocd         argocd-server             Synced                        serviceaccount/argocd-server configured
2020-06-04T17:18:02+02:00  rbac.authorization.k8s.io  ClusterRoleBinding            argocd         argocd-server            Running    Synced              clusterrolebinding.rbac.authorization.k8s.io/argocd-server reconciled. reconciliation required update. clusterrolebinding.rbac.authorization.k8s.io/argocd-server configured
2020-06-04T17:18:02+02:00  rbac.authorization.k8s.io  ClusterRoleBinding            argocd  argocd-application-controller   Running    Synced              clusterrolebinding.rbac.authorization.k8s.io/argocd-application-controller reconciled. reconciliation required update. clusterrolebinding.rbac.authorization.k8s.io/argocd-application-controller configured
2020-06-04T17:18:02+02:00                                Service                    argocd        argocd-metrics           OutOfSync  Healthy              service/argocd-metrics configured
2020-06-04T17:18:02+02:00                                Service                    argocd         argocd-server           OutOfSync  Healthy              service/argocd-server configured
2020-06-04T17:18:02+02:00   apps                      Deployment                    argocd         argocd-server           OutOfSync  Healthy              deployment.apps/argocd-server configured
2020-06-04T17:18:02+02:00                              ConfigMap                    argocd             argocd-cm             Synced                        configmap/argocd-cm configured
2020-06-04T17:18:02+02:00                             ServiceAccount                argocd     argocd-dex-server             Synced                        serviceaccount/argocd-dex-server configured
2020-06-04T17:18:02+02:00   apps                      Deployment                    argocd    argocd-repo-server           OutOfSync  Healthy              deployment.apps/argocd-repo-server configured
2020-06-04T17:18:02+02:00  apiextensions.k8s.io       CustomResourceDefinition      argocd  appprojects.argoproj.io         Running    Synced              customresourcedefinition.apiextensions.k8s.io/appprojects.argoproj.io unchanged
2020-06-04T17:18:02+02:00  rbac.authorization.k8s.io  RoleBinding                   argocd  argocd-application-controller    Synced                        rolebinding.rbac.authorization.k8s.io/argocd-application-controller reconciled. reconciliation required update. rolebinding.rbac.authorization.k8s.io/argocd-application-controller configured
2020-06-04T17:18:02+02:00                                Service                    argocd    argocd-repo-server           OutOfSync  Healthy              service/argocd-repo-server configured
2020-06-04T17:18:02+02:00   apps                      Deployment                    argocd  argocd-application-controller  OutOfSync  Healthy              deployment.apps/argocd-application-controller configured
2020-06-04T17:18:02+02:00            Service      argocd    argocd-repo-server             Synced  Healthy              service/argocd-repo-server configured
2020-06-04T17:18:02+02:00   apps  Deployment      argocd         argocd-server             Synced  Healthy              deployment.apps/argocd-server configured
2020-06-04T17:18:02+02:00   apps  Deployment      argocd          argocd-redis             Synced  Healthy              deployment.apps/argocd-redis configured
2020-06-04T17:18:02+02:00   apps  Deployment      argocd    argocd-repo-server             Synced  Healthy              deployment.apps/argocd-repo-server configured
2020-06-04T17:18:02+02:00   apps  Deployment      argocd     argocd-dex-server             Synced  Healthy              deployment.apps/argocd-dex-server configured
2020-06-04T17:18:02+02:00            Service      argocd          argocd-redis             Synced  Healthy              service/argocd-redis configured
2020-06-04T17:18:02+02:00            Service      argocd         argocd-server             Synced  Healthy              service/argocd-server configured
2020-06-04T17:18:02+02:00   apps  Deployment      argocd  argocd-application-controller    Synced  Healthy              deployment.apps/argocd-application-controller configured
2020-06-04T17:18:02+02:00            Service      argocd        argocd-metrics             Synced  Healthy              service/argocd-metrics configured
2020-06-04T17:18:02+02:00            Service      argocd  argocd-server-metrics            Synced  Healthy              service/argocd-server-metrics configured
2020-06-04T17:18:02+02:00            Service      argocd     argocd-dex-server             Synced  Healthy              service/argocd-dex-server configured

Name:               argocd
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          argocd
URL:                https://argocd.microk8s.local/applications/argocd
Repo:               https://github.com/dgo19/microk8s-walkthrough.git
Target:             HEAD
Path:               argocd
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        Synced to HEAD (e33892a)
Health Status:      Healthy

Operation:          Sync
Sync Revision:      e33892a088411735c9ae5ca3881f3d6e39da15cb
Phase:              Succeeded
Start:              2020-06-04 17:17:58 +0200 CEST
Finished:           2020-06-04 17:18:01 +0200 CEST
Duration:           3s
Message:            successfully synced (all tasks run)

GROUP                      KIND                      NAMESPACE  NAME                           STATUS   HEALTH   HOOK  MESSAGE
                           Namespace                 argocd     argocd                         Running  Synced         namespace/argocd configured
                           Secret                    argocd     argocd-secret                  Synced                  secret/argocd-secret configured
                           ConfigMap                 argocd     argocd-cm                      Synced                  configmap/argocd-cm configured
                           ConfigMap                 argocd     argocd-tls-certs-cm            Synced                  configmap/argocd-tls-certs-cm configured
                           ConfigMap                 argocd     argocd-rbac-cm                 Synced                  configmap/argocd-rbac-cm configured
                           ConfigMap                 argocd     argocd-ssh-known-hosts-cm      Synced                  configmap/argocd-ssh-known-hosts-cm configured
                           ServiceAccount            argocd     argocd-server                  Synced                  serviceaccount/argocd-server configured
                           ServiceAccount            argocd     argocd-application-controller  Synced                  serviceaccount/argocd-application-controller configured
                           ServiceAccount            argocd     argocd-dex-server              Synced                  serviceaccount/argocd-dex-server configured
apiextensions.k8s.io       CustomResourceDefinition  argocd     appprojects.argoproj.io        Running  Synced         customresourcedefinition.apiextensions.k8s.io/appprojects.argoproj.io unchanged
apiextensions.k8s.io       CustomResourceDefinition  argocd     applications.argoproj.io       Running  Synced         customresourcedefinition.apiextensions.k8s.io/applications.argoproj.io unchanged
rbac.authorization.k8s.io  ClusterRole               argocd     argocd-server                  Running  Synced         clusterrole.rbac.authorization.k8s.io/argocd-server reconciled. reconciliation required update. clusterrole.rbac.authorization.k8s.io/argocd-server configured
rbac.authorization.k8s.io  ClusterRole               argocd     argocd-application-controller  Running  Synced         clusterrole.rbac.authorization.k8s.io/argocd-application-controller reconciled. reconciliation required update. clusterrole.rbac.authorization.k8s.io/argocd-application-controller configured
rbac.authorization.k8s.io  ClusterRoleBinding        argocd     argocd-server                  Running  Synced         clusterrolebinding.rbac.authorization.k8s.io/argocd-server reconciled. reconciliation required update. clusterrolebinding.rbac.authorization.k8s.io/argocd-server configured
rbac.authorization.k8s.io  ClusterRoleBinding        argocd     argocd-application-controller  Running  Synced         clusterrolebinding.rbac.authorization.k8s.io/argocd-application-controller reconciled. reconciliation required update. clusterrolebinding.rbac.authorization.k8s.io/argocd-application-controller configured
rbac.authorization.k8s.io  Role                      argocd     argocd-application-controller  Synced                  role.rbac.authorization.k8s.io/argocd-application-controller reconciled. reconciliation required update. role.rbac.authorization.k8s.io/argocd-application-controller configured
rbac.authorization.k8s.io  Role                      argocd     argocd-dex-server              Synced                  role.rbac.authorization.k8s.io/argocd-dex-server reconciled. reconciliation required update. role.rbac.authorization.k8s.io/argocd-dex-server configured
rbac.authorization.k8s.io  Role                      argocd     argocd-server                  Synced                  role.rbac.authorization.k8s.io/argocd-server reconciled. reconciliation required update. role.rbac.authorization.k8s.io/argocd-server configured
rbac.authorization.k8s.io  RoleBinding               argocd     argocd-server                  Synced                  rolebinding.rbac.authorization.k8s.io/argocd-server reconciled. reconciliation required update. rolebinding.rbac.authorization.k8s.io/argocd-server configured
rbac.authorization.k8s.io  RoleBinding               argocd     argocd-application-controller  Synced                  rolebinding.rbac.authorization.k8s.io/argocd-application-controller reconciled. reconciliation required update. rolebinding.rbac.authorization.k8s.io/argocd-application-controller configured
rbac.authorization.k8s.io  RoleBinding               argocd     argocd-dex-server              Synced                  rolebinding.rbac.authorization.k8s.io/argocd-dex-server reconciled. reconciliation required update. rolebinding.rbac.authorization.k8s.io/argocd-dex-server configured
                           Service                   argocd     argocd-redis                   Synced   Healthy        service/argocd-redis configured
                           Service                   argocd     argocd-metrics                 Synced   Healthy        service/argocd-metrics configured
                           Service                   argocd     argocd-server-metrics          Synced   Healthy        service/argocd-server-metrics configured
                           Service                   argocd     argocd-repo-server             Synced   Healthy        service/argocd-repo-server configured
                           Service                   argocd     argocd-server                  Synced   Healthy        service/argocd-server configured
                           Service                   argocd     argocd-dex-server              Synced   Healthy        service/argocd-dex-server configured
apps                       Deployment                argocd     argocd-server                  Synced   Healthy        deployment.apps/argocd-server configured
apps                       Deployment                argocd     argocd-redis                   Synced   Healthy        deployment.apps/argocd-redis configured
apps                       Deployment                argocd     argocd-dex-server              Synced   Healthy        deployment.apps/argocd-dex-server configured
apps                       Deployment                argocd     argocd-application-controller  Synced   Healthy        deployment.apps/argocd-application-controller configured
apps                       Deployment                argocd     argocd-repo-server             Synced   Healthy        deployment.apps/argocd-repo-server configured
extensions                 Ingress                   argocd     ingress-argocd-server          Synced   Healthy        ingress.extensions/ingress-argocd-server configured
                           Namespace                            argocd                         Synced                  
apiextensions.k8s.io       CustomResourceDefinition             applications.argoproj.io       Synced                  
apiextensions.k8s.io       CustomResourceDefinition             appprojects.argoproj.io        Synced                  
rbac.authorization.k8s.io  ClusterRole                          argocd-application-controller  Synced                  
rbac.authorization.k8s.io  ClusterRole                          argocd-server                  Synced                  
rbac.authorization.k8s.io  ClusterRoleBinding                   argocd-application-controller  Synced                  
rbac.authorization.k8s.io  ClusterRoleBinding                   argocd-server                  Synced
```
```
$ argocd app list
NAME    CLUSTER                         NAMESPACE  PROJECT  STATUS  HEALTH   SYNCPOLICY  CONDITIONS  REPO                                               PATH    TARGET
argocd  https://kubernetes.default.svc  argocd     default  Synced  Healthy  <none>      <none>      https://github.com/dgo19/microk8s-walkthrough.git  argocd  HEAD
```


