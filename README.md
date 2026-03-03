# example-jenkins-kubernetes-deployment
A project for testing best practices when building and deploying  Jenkins with kubernetes.


## Installing Jenkins
First ensure helm is installed on your system.
Then run helm commands for adding jenkins helm charts
```
helm repo add jenkins https://charts.jenkins.io
helm repo update
```

after installing helm, we can install jenkins into the jenkins-controller namespace.
We start by setting up basic RBAC for jenkins with the helm/jenkins-namespaces-and-rbac.yaml file.
Then run the actual jenkins helm file, targeting the jenkins-controller namespace.

``` 
kubectl apply -f helm/jenkins-namespaces-and-rbac.yaml
helm upgrade --install jenkins jenkins/jenkins -n jenkins-controller -f helm/jenkins-values.yaml
```

