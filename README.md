# example-jenkins-kubernetes-deployment
The intent of this project is to build a simple DevOps system on a local linux server that follows best security and management practices.
This means proper kubernetes rbac permissions for jenkins and jenkins agents and a proper artifact storage source, in this case Harbor.
Harbor would ideally have an external artifact storage source, but for simplicity its all in the same kubernetes node.

I tested this setup on my Rockchip ROC-RK3588S-PC from Firefly, using Joshua Reik's Ubuntu server for Linux 6.1 as the OS.
It is an arm64 based system.

## Prerequisites
1. Kubernetes (k3s) - k3s specifically because it has the traefik ingress controller that the helm files use to expose access
2. Helm

I also recommend **k9s** or another UI for Kubernetes to hgelp with visualization and debugging.


## Installing Jenkins
Installing jenkins only requires 3 steps.
1. Update helm to get access to jenkins official helm charts:
```
helm repo add jenkins https://charts.jenkins.io
helm repo update
```
2. Apply the jenkins RBAC and configuration files:

```
kubectl apply -f helm/jenkins-namespaces-and-rbac.yaml
```

The rbac file creates the proper namespace for jenkins and creates a service role that jenkins will use to setup and deploy agents. (currently WIP)

3. Install Jenkins via the helm chart:
```
helm upgrade --install jenkins jenkins/jenkins -n jenkins-controller -f helm/jenkins-values.yaml
```


