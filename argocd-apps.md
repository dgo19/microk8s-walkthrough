# Argo CD - Applications
Deploy root application:
```
$ cd microk8s-walkthrough/applications/
$ kubectl apply -f applications.yaml
application.argoproj.io/applications created
```
Review diff and sync applications.
```
$ argocd app list
NAME          CLUSTER                         NAMESPACE  PROJECT  STATUS     HEALTH   SYNCPOLICY  CONDITIONS  REPO                                               PATH          TARGET
applications  https://kubernetes.default.svc  argocd     default  OutOfSync  Missing  <none>      <none>      https://github.com/dgo19/microk8s-walkthrough.git  applications  HEAD
argocd        https://kubernetes.default.svc  argocd     default  Synced     Healthy  <none>      <none>      https://github.com/dgo19/microk8s-walkthrough.git  argocd        HEAD
```
```
$ argocd app diff applications
===== argoproj.io/Application argocd/mariadb-pv ======
0a1,17
> apiVersion: argoproj.io/v1alpha1
> kind: Application
> metadata:
>   labels:
>     app.kubernetes.io/instance: applications
>   name: mariadb-pv
>   namespace: argocd
> spec:
>   destination:
>     namespace: mariadb-pv
>     server: https://kubernetes.default.svc
>   project: default
>   source:
>     path: applications/mariadb-pv
>     repoURL: https://github.com/dgo19/microk8s-walkthrough.git
>     targetRevision: HEAD
>   syncPolicy: {}
===== argoproj.io/Application argocd/monitoring ======
0a1,17
> apiVersion: argoproj.io/v1alpha1
> kind: Application
> metadata:
>   labels:
>     app.kubernetes.io/instance: applications
>   name: monitoring
>   namespace: argocd
> spec:
>   destination:
>     namespace: monitoring
>     server: https://kubernetes.default.svc
>   project: default
>   source:
>     path: applications/monitoring
>     repoURL: https://github.com/dgo19/microk8s-walkthrough.git
>     targetRevision: HEAD
>   syncPolicy: {}
===== argoproj.io/Application argocd/postgres-operator ======
0a1,17
> apiVersion: argoproj.io/v1alpha1
> kind: Application
> metadata:
>   labels:
>     app.kubernetes.io/instance: applications
>   name: postgres-operator
>   namespace: argocd
> spec:
>   destination:
>     namespace: postgres-operator
>     server: https://kubernetes.default.svc
>   project: default
>   source:
>     path: applications/postgres-operator
>     repoURL: https://github.com/dgo19/microk8s-walkthrough.git
>     targetRevision: HEAD
>   syncPolicy: {}
 ... output omitted ...
```
```
$ argocd app sync applications
TIMESTAMP                  GROUP              KIND    NAMESPACE                  NAME    STATUS    HEALTH        HOOK  MESSAGE
2020-06-04T17:35:24+02:00  argoproj.io  Application      argocd            monitoring  OutOfSync  Missing              
2020-06-04T17:35:24+02:00  argoproj.io  Application      argocd     postgres-operator  OutOfSync  Missing              
2020-06-04T17:35:24+02:00  argoproj.io  Application      argocd     test-my-webserver  OutOfSync  Missing              
2020-06-04T17:35:24+02:00  argoproj.io  Application      argocd           zabbix-test  OutOfSync  Missing              
2020-06-04T17:35:24+02:00  argoproj.io  Application      argocd            configmaps  OutOfSync  Missing              
2020-06-04T17:35:24+02:00  argoproj.io  Application      argocd         ingress-nginx  OutOfSync  Missing              
2020-06-04T17:35:24+02:00  argoproj.io  Application      argocd     mariadb-ephemeral  OutOfSync  Missing              
2020-06-04T17:35:24+02:00  argoproj.io  Application      argocd            mariadb-pv  OutOfSync  Missing              
2020-06-04T17:35:25+02:00  argoproj.io  Application      argocd         ingress-nginx    Synced  Missing              
2020-06-04T17:35:25+02:00  argoproj.io  Application      argocd            monitoring    Synced  Missing              
2020-06-04T17:35:25+02:00  argoproj.io  Application      argocd     test-my-webserver    Synced  Missing              
2020-06-04T17:35:25+02:00  argoproj.io  Application      argocd     mariadb-ephemeral    Synced  Missing              
2020-06-04T17:35:25+02:00  argoproj.io  Application      argocd            mariadb-pv    Synced  Missing              
2020-06-04T17:35:25+02:00  argoproj.io  Application      argocd            configmaps  OutOfSync  Missing              application.argoproj.io/configmaps created
2020-06-04T17:35:25+02:00  argoproj.io  Application      argocd         ingress-nginx    Synced   Missing              application.argoproj.io/ingress-nginx created
2020-06-04T17:35:25+02:00  argoproj.io  Application      argocd     test-my-webserver    Synced   Missing              application.argoproj.io/test-my-webserver created
2020-06-04T17:35:25+02:00  argoproj.io  Application      argocd            mariadb-pv    Synced   Missing              application.argoproj.io/mariadb-pv created
2020-06-04T17:35:25+02:00  argoproj.io  Application      argocd            monitoring    Synced   Missing              application.argoproj.io/monitoring created
2020-06-04T17:35:25+02:00  argoproj.io  Application      argocd     mariadb-ephemeral    Synced   Missing              application.argoproj.io/mariadb-ephemeral created
2020-06-04T17:35:25+02:00  argoproj.io  Application      argocd           zabbix-test  OutOfSync  Missing              application.argoproj.io/zabbix-test created
2020-06-04T17:35:25+02:00  argoproj.io  Application      argocd     postgres-operator  OutOfSync  Missing              application.argoproj.io/postgres-operator created
2020-06-04T17:35:25+02:00  argoproj.io  Application      argocd           zabbix-test    Synced  Missing              application.argoproj.io/zabbix-test created
2020-06-04T17:35:25+02:00  argoproj.io  Application      argocd     postgres-operator    Synced  Missing              application.argoproj.io/postgres-operator created
2020-06-04T17:35:25+02:00  argoproj.io  Application      argocd            configmaps    Synced  Missing              application.argoproj.io/configmaps created

Name:               applications
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          argocd
URL:                https://argocd.microk8s.local/applications/applications
Repo:               https://github.com/dgo19/microk8s-walkthrough.git
Target:             HEAD
Path:               applications
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        Synced to HEAD (77f52aa)
Health Status:      Healthy

Operation:          Sync
Sync Revision:      77f52aa71a62f1260a7691b8cd41fed867b12c6c
Phase:              Succeeded
Start:              2020-06-04 17:35:24 +0200 CEST
Finished:           2020-06-04 17:35:25 +0200 CEST
Duration:           1s
Message:            successfully synced (all tasks run)

GROUP        KIND         NAMESPACE  NAME               STATUS  HEALTH  HOOK  MESSAGE
argoproj.io  Application  argocd     ingress-nginx      Synced                application.argoproj.io/ingress-nginx created
argoproj.io  Application  argocd     test-my-webserver  Synced                application.argoproj.io/test-my-webserver created
argoproj.io  Application  argocd     mariadb-pv         Synced                application.argoproj.io/mariadb-pv created
argoproj.io  Application  argocd     monitoring         Synced                application.argoproj.io/monitoring created
argoproj.io  Application  argocd     mariadb-ephemeral  Synced                application.argoproj.io/mariadb-ephemeral created
argoproj.io  Application  argocd     zabbix-test        Synced                application.argoproj.io/zabbix-test created
argoproj.io  Application  argocd     postgres-operator  Synced                application.argoproj.io/postgres-operator created
argoproj.io  Application  argocd     configmaps         Synced                application.argoproj.io/configmaps created
```
View the list of applications.
```
$ argocd app list
NAME               CLUSTER                         NAMESPACE          PROJECT  STATUS     HEALTH   SYNCPOLICY  CONDITIONS  REPO                                               PATH                            TARGET
applications       https://kubernetes.default.svc  argocd             default  Synced     Healthy  <none>      <none>      https://github.com/dgo19/microk8s-walkthrough.git  applications                    HEAD
argocd             https://kubernetes.default.svc  argocd             default  Synced     Healthy  <none>      <none>      https://github.com/dgo19/microk8s-walkthrough.git  argocd                          HEAD
configmaps         https://kubernetes.default.svc  configmaps         default  OutOfSync  Missing  <none>      <none>      https://github.com/dgo19/microk8s-walkthrough.git  applications/configmaps         HEAD
ingress-nginx      https://kubernetes.default.svc  ingress-nginx      default  OutOfSync  Healthy  <none>      <none>      https://github.com/dgo19/microk8s-walkthrough.git  applications/ingress-nginx      HEAD
mariadb-ephemeral  https://kubernetes.default.svc  mariadb-ephemeral  default  OutOfSync  Missing  <none>      <none>      https://github.com/dgo19/microk8s-walkthrough.git  applications/mariadb-ephemeral  HEAD
mariadb-pv         https://kubernetes.default.svc  mariadb-pv         default  OutOfSync  Missing  <none>      <none>      https://github.com/dgo19/microk8s-walkthrough.git  applications/mariadb-pv         HEAD
monitoring         https://kubernetes.default.svc  monitoring         default  OutOfSync  Missing  <none>      <none>      https://github.com/dgo19/microk8s-walkthrough.git  applications/monitoring         HEAD
postgres-operator  https://kubernetes.default.svc  postgres-operator  default  OutOfSync  Missing  <none>      <none>      https://github.com/dgo19/microk8s-walkthrough.git  applications/postgres-operator  HEAD
test-my-webserver  https://kubernetes.default.svc  test-my-webserver  default  OutOfSync  Missing  <none>      <none>      https://github.com/dgo19/microk8s-walkthrough.git  applications/test-my-webserver  HEAD
zabbix-test        https://kubernetes.default.svc  zabbix-test        default  OutOfSync  Missing  <none>      <none>      https://github.com/dgo19/microk8s-walkthrough.git  applications/zabbix-test        HEAD
```


