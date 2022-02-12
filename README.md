# microk8s walkthrough (archived)

# This project has been archived.
# For the new version with k3s, see [github.com/dgo19/k3s-demo](https://github.com/dgo19/k3s-demo/)

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
