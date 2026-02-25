# Set up Kubernetes
This file describes how to take a basic kubernetes cluster and set it up with proper permissions for jenkins use.
This assumes you are running all commands on linux/Unix.
Window users should consider using wsl for this.


## Create the kubernetes cluster
Create a kubernetes cluster using any number of the available tools, such as docker desktop

## Create Jenkins-agent Namespace
start by running the command: `kubectl apply -f jenkins-rbac.yml`
This command will create the jenkins-agent namespace and setup the jenkins-agent role, which jenkins can use when provisioning agents in kubernetes


## Setup kubernetes in jenkins
After creating the jenkins-agent namespace, 
1. Go to Jenkins > Manange Jenkins > Plugins, and make sure the Kubernetes plugin is installed
2. Go to Jenkins > Manage Jenkins > Clouds, and create a new kubernetes cloud as described here: https://plugins.jenkins.io/kubernetes/
make sure the kubernetes namespace is set to jenkins-agent
you can confirm it works with the test credentials button

## Setup docker registry
Now that kubernetes is set up, we want to setup our docker registry to store our containers.
You can also use an existing source like dockerhub, but for testing purposes - and so we dont push a ton of images to dockerhub - we are setting up a private authenticated registry.
Once we are ready to push to dockerhub, its just a matter of changing the docker config file to point to the registry we want.

To handle this, we need to create our docker registry credentials and save them in the auth file, then load them on registry setup.
1. Save your login credentials with the command (change 'testuser' and 'testpassword' to your credentials)
```
USERNAME=testuser
PASSWORD=testpassword
mkdir -p ~/docker-registry/auth
docker run --entrypoint htpasswd httpd:2 -Bbn $USERNAME $PASSWORD > ~/docker-registry/auth/htpasswd
```
Then create the registry with this command
```
docker run -d -p 5000:5000 --name local-auth-registry --restart=always -v $HOME/docker-registry/auth:/auth -e "REGISTRY_AUTH=htpasswd" -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" -e "REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd" registry:2
```
this will create a docker registry with authentication, that can be accessed on port 5000.

to ensure it works, run `docker login host.docker.internal:5000` and input your credentials when prompted.

## Setup kubernetes access to docker registry
Now that we have both kubernetes and the docker registry setup, we need to give kubernetes access to the docker registry.
The recommended way, at least for private registries, is to add a docker-registry secret in the kubernetes cluster. 

Make sure you replace the credentials with your actual registry credentials.
```
USERNAME=testuser
PASSWORD=testpassword

kubectl create secret docker-registry docker-registry-creds \
    --docker-server=host.docker.internal:5000 \
    --docker-username=$USERNAME \
    --docker-password=$PASSWORD \
    --namespace jenkins-agent
```
this will create the secret that your jenkins-agent account will now have access to. Notice how in the jenkins agent 'kaniko-builder.yml' file, we have a volume that pulls the docker-registry-creds secret. That will become the auth config for kaniko to push to the registry.

