# microk8s walkthrough

## Requirements
- ubuntu or linux distributrion with snap
- addition packages
  - curl
  - git
  - snapd (on debian)

[Installation of microk8s](install-microk8s-addons.md)

## App Deployments
- Pods, Deployment, Service, Ingress
  - [nginx webserver example](test-my-webserver.md)
- ConfigMaps
  - [nginx webserver with configmaps](configmap-webserver.md)
- ArgoCD
  - [ArgoCD Deployment and first steps](argocd.md)
  - [ArgoCD Application Deployment](argocd-apps.md)
- Monitoring
  - [Monitoring (Grafana, Prometheus, Alertmanager, Node-Exporter)](prometheus.md)
