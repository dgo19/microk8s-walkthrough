# Prometheus Monitoring
Verify, if kubelet is started with "--authentication-token-webhook" in args file "/var/snap/microk8s/current/args/kubelet"
If not, add the argument and restart kubelet.
```
$ echo "--authentication-token-webhook" >> /var/snap/microk8s/current/args/kubelet
$ sudo snap restart microk8s.daemon-kubelet
```
Activate the microk8s prometheus addon.
```
$ microk8s enable prometheus
Enabling Prometheus
namespace/monitoring created
customresourcedefinition.apiextensions.k8s.io/alertmanagers.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/podmonitors.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/prometheuses.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/prometheusrules.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/servicemonitors.monitoring.coreos.com created
clusterrole.rbac.authorization.k8s.io/prometheus-operator created
clusterrolebinding.rbac.authorization.k8s.io/prometheus-operator created
deployment.apps/prometheus-operator created
service/prometheus-operator created
serviceaccount/prometheus-operator created
alertmanager.monitoring.coreos.com/main created
secret/alertmanager-main created
service/alertmanager-main created
serviceaccount/alertmanager-main created
servicemonitor.monitoring.coreos.com/alertmanager created
secret/grafana-datasources created
configmap/grafana-dashboard-apiserver created
configmap/grafana-dashboard-cluster-total created
configmap/grafana-dashboard-controller-manager created
configmap/grafana-dashboard-k8s-resources-cluster created
configmap/grafana-dashboard-k8s-resources-namespace created
configmap/grafana-dashboard-k8s-resources-node created
configmap/grafana-dashboard-k8s-resources-pod created
configmap/grafana-dashboard-k8s-resources-workload created
configmap/grafana-dashboard-k8s-resources-workloads-namespace created
configmap/grafana-dashboard-kubelet created
configmap/grafana-dashboard-namespace-by-pod created
configmap/grafana-dashboard-namespace-by-workload created
configmap/grafana-dashboard-node-cluster-rsrc-use created
configmap/grafana-dashboard-node-rsrc-use created
configmap/grafana-dashboard-nodes created
configmap/grafana-dashboard-persistentvolumesusage created
configmap/grafana-dashboard-pod-total created
configmap/grafana-dashboard-pods created
configmap/grafana-dashboard-prometheus-remote-write created
configmap/grafana-dashboard-prometheus created
configmap/grafana-dashboard-proxy created
configmap/grafana-dashboard-scheduler created
configmap/grafana-dashboard-statefulset created
configmap/grafana-dashboard-workload-total created
configmap/grafana-dashboards created
deployment.apps/grafana created
service/grafana created
serviceaccount/grafana created
servicemonitor.monitoring.coreos.com/grafana created
clusterrole.rbac.authorization.k8s.io/kube-state-metrics created
clusterrolebinding.rbac.authorization.k8s.io/kube-state-metrics created
deployment.apps/kube-state-metrics created
role.rbac.authorization.k8s.io/kube-state-metrics created
rolebinding.rbac.authorization.k8s.io/kube-state-metrics created
service/kube-state-metrics created
serviceaccount/kube-state-metrics created
servicemonitor.monitoring.coreos.com/kube-state-metrics created
clusterrole.rbac.authorization.k8s.io/node-exporter created
clusterrolebinding.rbac.authorization.k8s.io/node-exporter created
daemonset.apps/node-exporter created
service/node-exporter created
serviceaccount/node-exporter created
servicemonitor.monitoring.coreos.com/node-exporter created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
clusterrole.rbac.authorization.k8s.io/prometheus-adapter created
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrolebinding.rbac.authorization.k8s.io/prometheus-adapter created
clusterrolebinding.rbac.authorization.k8s.io/resource-metrics:system:auth-delegator created
clusterrole.rbac.authorization.k8s.io/resource-metrics-server-resources created
configmap/adapter-config created
deployment.apps/prometheus-adapter created
rolebinding.rbac.authorization.k8s.io/resource-metrics-auth-reader created
service/prometheus-adapter created
serviceaccount/prometheus-adapter created
clusterrole.rbac.authorization.k8s.io/prometheus-k8s created
clusterrolebinding.rbac.authorization.k8s.io/prometheus-k8s created
servicemonitor.monitoring.coreos.com/prometheus-operator created
prometheus.monitoring.coreos.com/k8s created
rolebinding.rbac.authorization.k8s.io/prometheus-k8s-config created
rolebinding.rbac.authorization.k8s.io/prometheus-k8s created
rolebinding.rbac.authorization.k8s.io/prometheus-k8s created
rolebinding.rbac.authorization.k8s.io/prometheus-k8s created
role.rbac.authorization.k8s.io/prometheus-k8s-config created
role.rbac.authorization.k8s.io/prometheus-k8s created
role.rbac.authorization.k8s.io/prometheus-k8s created
role.rbac.authorization.k8s.io/prometheus-k8s created
prometheusrule.monitoring.coreos.com/prometheus-k8s-rules created
service/prometheus-k8s created
serviceaccount/prometheus-k8s created
servicemonitor.monitoring.coreos.com/prometheus created
servicemonitor.monitoring.coreos.com/kube-apiserver created
servicemonitor.monitoring.coreos.com/coredns created
servicemonitor.monitoring.coreos.com/kube-controller-manager created
servicemonitor.monitoring.coreos.com/kube-scheduler created
servicemonitor.monitoring.coreos.com/kubelet created
The Prometheus operator is enabled (user/pass: admin/admin)
```
Deploy ingress for prometheus, alertmanager, grafana by Argo CD or by applying the yaml files from "microk8s-walkthrough/applications/monitoring"
```
$ argocd app get monitoring
Name:               monitoring
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          monitoring
URL:                https://argocd.microk8s.local/applications/monitoring
Repo:               https://github.com/dgo19/microk8s-walkthrough.git
Target:             HEAD
Path:               applications/monitoring
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        OutOfSync from HEAD (c0344f9)
Health Status:      Missing

GROUP       KIND     NAMESPACE   NAME          STATUS     HEALTH   HOOK  MESSAGE
extensions  Ingress  monitoring  alertmanager  OutOfSync  Missing        
extensions  Ingress  monitoring  grafana       OutOfSync  Missing        
extensions  Ingress  monitoring  prometheus    OutOfSync  Missing        
```
```
$ argocd app sync monitoring
TIMESTAMP                  GROUP             KIND   NAMESPACE                  NAME    STATUS    HEALTH        HOOK  MESSAGE
2020-06-05T15:11:16+02:00  extensions     Ingress  monitoring               grafana  OutOfSync  Missing              
2020-06-05T15:11:16+02:00  extensions     Ingress  monitoring            prometheus  OutOfSync  Missing              
2020-06-05T15:11:16+02:00  extensions     Ingress  monitoring          alertmanager  OutOfSync  Missing              
2020-06-05T15:11:17+02:00  extensions     Ingress  monitoring          alertmanager    Synced  Progressing              
2020-06-05T15:11:17+02:00  extensions     Ingress  monitoring               grafana    Synced  Progressing              
2020-06-05T15:11:17+02:00  extensions     Ingress  monitoring          alertmanager    Synced   Progressing              ingress.extensions/alertmanager created
2020-06-05T15:11:17+02:00  extensions     Ingress  monitoring               grafana    Synced   Progressing              ingress.extensions/grafana created
2020-06-05T15:11:17+02:00  extensions     Ingress  monitoring            prometheus  OutOfSync  Missing                  ingress.extensions/prometheus created

Name:               monitoring
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          monitoring
URL:                https://argocd.microk8s.local/applications/monitoring
Repo:               https://github.com/dgo19/microk8s-walkthrough.git
Target:             HEAD
Path:               applications/monitoring
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        Synced to HEAD (c0344f9)
Health Status:      Progressing

Operation:          Sync
Sync Revision:      c0344f959ee4b18fb4ec3af5f1497a4504d400e3
Phase:              Succeeded
Start:              2020-06-05 15:11:16 +0200 CEST
Finished:           2020-06-05 15:11:17 +0200 CEST
Duration:           1s
Message:            successfully synced (all tasks run)

GROUP       KIND     NAMESPACE   NAME          STATUS  HEALTH       HOOK  MESSAGE
extensions  Ingress  monitoring  alertmanager  Synced  Progressing        ingress.extensions/alertmanager created
extensions  Ingress  monitoring  grafana       Synced  Progressing        ingress.extensions/grafana created
extensions  Ingress  monitoring  prometheus    Synced  Progressing        ingress.extensions/prometheus created
```
```
$ argocd app get monitoring
Name:               monitoring
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          monitoring
URL:                https://argocd.microk8s.local/applications/monitoring
Repo:               https://github.com/dgo19/microk8s-walkthrough.git
Target:             HEAD
Path:               applications/monitoring
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        Synced to HEAD (c0344f9)
Health Status:      Healthy

GROUP       KIND     NAMESPACE   NAME          STATUS  HEALTH   HOOK  MESSAGE
extensions  Ingress  monitoring  alertmanager  Synced  Healthy        ingress.extensions/alertmanager created
extensions  Ingress  monitoring  grafana       Synced  Healthy        ingress.extensions/grafana created
extensions  Ingress  monitoring  prometheus    Synced  Healthy        ingress.extensions/prometheus created
```
Explore the webui.
- Grafana [https://grafana.microk8s.local/](https://grafana.microk8s.local/)
- Alertmanager [https://alertmanager.microk8s.local/](https://alertmanager.microk8s.local/)
- Prometheus [https://prometheus.microk8s.local/](https://prometheus.microk8s.local/)

