# AspnetDevOps

Deploying .Net Microservices with K8s, AKS and Azure DevOps

Article with instructions for creating 3 container application and deploy to AKS on Azure:
[Preparing Multi-Container Microservices Applications for Deployment](https://medium.com/aspnetrun/preparing-multi-container-microservices-applications-for-deployment-793d60f48d31)

## deploy .net microservices with k8s, aks and azure devops

### prerequisites and source code

source code: https://github.com/aspnetrun/run-devops

tools:

- visual studio 2019
  develop app
  dockerize app

- vs code
  writing kubernetes manifest files
  other kubernetes manifest files
  deploying containers on kubernetes clusters

Docker Desktop
Docker Account for pushing images to Docker Hub

Azure Subscription for creating all Azure resources

Azure DevOps Account for ci/cd devops pipelines

## Docker Compose

NOTE: REBUILD IMAGES TO INCLUDE CODE CHANGES AND START
docker-compose -f docker-compose.yml -f docker-compose.override.yml up --build
NOTE: START CONTAINERS FROM EXISTING IMAGES WITHOUT REBUILDING
docker-compose -f docker-compose.yml -f docker-compose.override.yml up -d
NOTE: STOP RUNNING CONTAINERS AND REMOVE CONTAINERS
docker-compose -f docker-compose.yml -f docker-compose.override.yml down

NO CHANGES TO CODE USE -d
docker-compose -f .\docker-compose.yml -f .\docker-compose.override.yml up -d

ANY CHANGES TO CODE USE --build
docker-compose -f .\docker-compose.yml -f .\docker-compose.override.yml up --build

docker-compose down

## Docker Commands

docker volume prune

## Docker Hub

from vs you can login in powershell > docker login
also you can use vs code extension to access registry

tag image:
right click container > shoppingclient:latest > tag > jpcassidy/shoppingapp:latest

push image:
docker push jpcassidy/shoppingapp:latest
or > right click image > push

## Azure

Shopping.Client

Resource Group - shoppingapp

App - shoppingappclient.azurewebsites.net

## Mongo

pull mongo db from hub.docker.com/\_/mongo

> docker pull mongo

run mongo

> docker run -d -p 27017:27017 --name shopping-mongo mongo

logs

> docker logs -f shopping-mongo

execute interactive

> docker exec -it shopping-mongo /bin/bash
> ls > show list of files/folders
> mongo > start mongo cmd
> show dbs > list of databases > show databases
> use ProductDb > create and switch to CatalogDb
> db.createCollection('Products')
> db.Products.insertMany([{'Name': 'Asus Laptop','Category': 'Computers', 'Summary': 'Summary', 'Description': 'Description', 'ImageFile': 'ImageFile', 'Price':1999.99 }, {'Name': 'HP Laptop','Category': 'Computers', 'Summary': 'Summary', 'Description': 'Description', 'ImageFile': 'ImageFile', 'Price':1205.99 }])
> db.Products.find({}).pretty() > get all
> db.Products.remove({}) > remove all

## Kubernetes

[Kubernetes Cheatsheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

### Setup Dashboard

Running Kubernetes Dashboard - instructions provided by Andre Lock
[https://andrewlock.net/running-kubernetes-and-the-dashboard-with-docker-desktop/](https://andrewlock.net/running-kubernetes-and-the-dashboard-with-docker-desktop/)

To install the Kubernetes Dashboard, open terminal and run following:

> kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.5.0/aio/deploy/recommended.yaml

#### Start/Login Kubernetes Dashboard

> kubectl proxy

Login > http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

#### Setup Login User w/token

Create a user to log into Kubernetes Dashboard > https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/

Instructions for creating a sample user
https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md

> create dashboard-adminuser.yaml
> kubectl apply -f dashboard-adminuser.yaml
> create dashboard-cluster-admin-role.yaml
> kubectl apply -f dashboard-cluster-admin-role.yaml

Get the Bearer Token

> kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"

Copy token and use it to log into kubernetes-dashboard

Alternate way to get Bearer Token

> kubectl describe secret -n kube-system

#### Disabling the login promtp in Kubernetes Dashboard

You can override the login page by running following command:

```powershell

kubectl patch deployment kubernetes-dashboard -n kubernetes-dashboard --type 'json' -p '[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--enable-skip-login"}]'

```

If you want to make this change manually, run the following, which opens notepad.exe:

```powershell
kubectl edit deployment kubernetes-dashboard -n kubernetes-dashboard
```

Add the --enable-skip-login argument, as shown here:

```yaml
spec:
  template:
    spec:
      containers:
        - args:
            - --auto-generate-certificates
            - --namespace=kubernetes-dashboard
            - --enable-skip-login # add this argument
          image: kubernetesui/dashboard:v2.5.0
```

#### Setup Metrics Server

[GitHub Releases](https://github.com/kubernetes-sigs/metrics-server/releases)

Run the following command to download the Metrics Server manifests and install them in your cluster:

> kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.6.1/components.yaml

Patch one of the deployments - add the --kubelet-insecure-tls argument to the metrics-server deployment, otherwise you'll see an error saying something like unable to fetch metrics from node docker-desktop. The following command patches the deployment:

> kubectl patch deployment metrics-server -n kube-system --type 'json' -p '[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--kubelet-insecure-tls"}]'

### Commands

kubectl
see organized commands

cluster commands:
what clusters are available: kubectl config get-contexts
what is current cluster: kubectl config current-context
switch cluster context: kubectl config use-context --help

general commands:
kubectl --help
kubectl version
kubectl cluster-info
kubectl get nodes
kubectl get pod
kubectl get services
kubectl get all
kubectl get all -- pods, services, deployments..

kubectl get pods --all-namespaces
kubectl get pods -o wide

---

Imperative - Declarative
Imperative Commands -- create and run resources

kubectl run [container_name] --image=[image_name]
kubectl port-forward [pod] [ports]

Declarative Commands -- create or modify resources
kubectl create [resource]
kubectl apply [resource]

---

kubectl run swn-nginx --image=nginx
kubectl get pods
kubectl get all

kubectl port-forward swn-nginx 8080:80
kubectl delete deployment swn-nginx
kubectl get pods --watch

---

kubectl create
-- there is no pod
so its abraction from deployment so we should create deployment

kubectl create deployment name --image=image [--dry-run] [options]
kubectl create deployment nginx-depl --image=nginx
kubectl get deployment
kubectl get pod
kubectl get replicaset
kubectl get all

kubectl edit deployment nginx-depl
kubectl delete deployment nginx-depl

---

yaml file has 3 parts

1: metadata

2: spec

3: kubernetes extends and adds to this file
spec -- add to spec
status
to see this deployment status file: kubectl get deployment [deployment-name] -o yaml

---

Declarative Commands

create a nginx-depl.yaml file

kubectl apply -f .\nginx-depl.yaml
see deployment by running: kubectl get all:

- pod
- replicaset
- deployment

kubectl create -f
kubectl edit -f
kubectl delete -f

kubectl delete -f .\nginx-depl.yaml

---

Connect to pod: kubectl exec
Historical info of pod: kubectl describe

---

Create Service on Kubernetes

nginx-service.yaml

kubectl apply -f .\nginx-service.yaml

---

apiVersion: v1
kind: Service
metadata:
name: nginx-service
spec:
selector:
app: nginx
ports: - portocol: TCP
port: 80
targetPort: 8080

---

kubectl apply -f .\nginx-service.yaml

once you apply your deployment and the pods are created, if the service references the app,
then run these commands to see ips and ports associated by service to pods:

kubectl describe service nginx-service
kubectl get pod -o wide

---

kubectl edit
-- edit deployment by running command:
kubectl edit deployment name
kubectl edit deployment nginx-depl
-- THIS WILL OPEN NOTEPAD IN WINDOWS OS

---

Debugging Pods

kubectl logs nginx-depl-5c8bf76b5b-tzv2k

-- create new depl - mongo
kubectl create deployment mongo-depl --image=mongo
kubectl get pod
kubectl describe pod mongo-depl-5fd6b7d4b4-6xzjd
kubectl logs mongo-depl-5fd6b7d4b4-6xzjd
kubectl exec mongo-depl-5fd6b7d4b4-6xzjd -it sh
ls
mongo
show dbs

-- delete reasource

kubectl get deployment
kubectl get replicaset

kubectl delete deployment nginx-depl
kubectl delete deployment mongo-depl

kubectl get pod --watch
kubectl get replicaset

---

## section 8: create kubernetes local environment

- Shopping.API yaml
- Shopping.Client yaml
- Mongo Db yaml
- deployment yaml file
- service yaml file
- config map
- secret definitions - storing database connection definitions

1 create mongo.yaml
2 create mongo-secret.yaml
2a base64 the secrets: https://www.base64encode.org/

> username=username=dXNlcm5hbWU=
> password=password=cGFzc3dvcmQ=

NOTE: you can also run the following command in wsl to generate the base64 encrypted secret:

> echo -n 'username' | base64
> echo -n 'password' | base64

2b apply mongo-secret.yaml > kubectl apply -f .\k8s\mongo-secret.yaml
2c check secret applied successfully > kubectl get secret
3 update secrects in mongo.yaml
3a apply mongo.yaml > kubectl apply -f .\k8s\mongo.yaml

> kubectl get pod --watch
> kubectl get all
> copy pod name and run: kubectl describe pod [pod name]

## section 8: Build Shopping Docker Images, Tag and Push to Docker Hub

1 build images and test containers

> docker-compose -f .\docker-compose.yml -f .\docker-compose.override.yml up --build
> docker-compose down

2 tag images

> docker tag [Image Id] jpcassidy/shoppingclient
> docker tag b644e980 jpcassidy/shoppingclient
> docker tag [Image Id] jpcassidy/shoppingapi
> docker tag 4ab0531 jpcassidy/shoppingapi
> docker tag [Image Id] jpcassidy/mongodb

3 push images to Docker Hub

> docker push jpcassidy/shoppingclient
> docker push jpcassidy/shoppingapi

## section 8: create Shopping.API yaml

1 create Shopping.API yaml
1a test Shopping.API yaml

> kubectl apply -f .\k8s\mongo-secret.yaml
> kubectl apply -f .\k8s\mongo.yaml
> kubectl apply -f .\k8s\shoppingapi.yaml
> kubectl get all

2 update connectionstring
2a create mongo-configmap.yaml
2b test > kubectl get cm

> kubectl apply -f .\k8s\mongo-configmap.yaml
> kubectl apply -f .\k8s\shoppingapi.yaml

## section 8: create Shopping.Client yaml

1 create Shopping.Client yaml

2a create shoppingapi-config.yaml
2b test > kubectl get cm

3 test Shopping.Client yaml

> kubectl apply -f .\k8s\shoppingapi-configmap.yaml
> kubectl apply -f .\k8s\shoppingclient.yaml

## section 8: clear and create all k8s resources on local cluster

1 clear all resources

> kubectl delete -f .\k8s\

2 apply all resources

> kubectl apply -f .\k8s\

## Section 9: Azure cli, AKS

Steps

1 docker build/push to Docker Registry (Docker Hub, ACR)
2 create deploy to cluster: Azure cli (kubectl create -f app-deploy.yaml)
3 configure external access (Azure Load Balancer)

### Azure cli commands

az version

az login

Create a resource group
az group create --name aspnetDevOps --location eastus

Create an Azure Container Registry
az acr create --resource-group aspnetDevOps --name aspnetdevopsshoppingacr --sku Basic

Enable Admin Account for ACR Pull
az acr update -n aspnetdevopsshoppingacr --admin-enabled true

Log in to the container registry
az acr login --name aspnetdevopsshoppingacr

### Tag a container image

get the login server address
az acr list --resource-group aspnetDevOps --query "[].{acrLoginServer:loginServer}" --output table
aspnetdevopsshoppingacr.azurecr.io

Tag your images

docker tag shoppingapi:latest aspnetdevopsshoppingacr.azurecr.io/shoppingapi:v1
docker tag shoppingclient:latest aspnetdevopsshoppingacr.azurecr.io/shoppingclient:v1

Push images to registry

docker push aspnetdevopsshoppingacr.azurecr.io/shoppingapi:v1
docker push aspnetdevopsshoppingacr.azurecr.io/shoppingclient:v1

List images in registry
az acr repository list --name aspnetdevopsshoppingacr --output table

See tags
az acr repository show-tags --name aspnetdevopsshoppingacr --repository shoppingapi --output table
az acr repository show-tags --name aspnetdevopsshoppingacr --repository shoppingclient --output table

### AKS Cluster

Create AKS cluster with attaching ACR
az aks create --resource-group aspnetDevOps --name aspnetDevOpsAKSCluster --node-count 1 --generate-ssh-keys --attach-acr aspnetdevopsshoppingacr

### az aks cli

Install the Kubernetes CLI
az aks install-cli

Connect to cluster using kubectl
az aks get-credentials --resource-group aspnetDevOps --name aspnetDevOpsAKSCluster
Merged "aspnetDevOpsAKSCluster" as current context in C:\Users\jpcas\.kube\config

Create PULL Secret

> kubectl create secret docker-registry acr-secret --docker-server=aspnetdevopsshoppingacr.azurecr.io --docker-username=[uername] --docker-password=[password] --docker-email=[email]

Create all aks yaml files

Confirm connected to AKS from command line

> kubectl config get-contexts

```text
CURRENT   NAME                     CLUSTER                  AUTHINFO                                          NAMESPACE

*         aspnetDevOpsAKSCluster   aspnetDevOpsAKSCluster   clusterUser_aspnetDevOps_aspnetDevOpsAKSCluster
          docker-desktop           docker-desktop           docker-desktop
```

check current context

> kubectl config current-context
> aspnetDevOpsAKSCluster

change context

> kubectl config use-context docker-desktop

#### Deploy microservices to AKS

> kubectl apply -f .\aks\

configmap/mongo-configmap created
secret/mongo-secret created
deployment.apps/mongo-deployment created
service/mongo-service created
configmap/shoppingapi-configmap created
deployment.apps/shoppingapi-deployment created
service/shoppingapi-service created
deployment.apps/shoppingclient-deployment created
service/shoppingclient-service created

> kubectl get all

Get API from service/shoppingclient-service
Open in browser

#### Debug Errors

Get Pod

> kubectl get pod

Copy Name

> kubectl describe pod [podname]

Review Event list and read Message for each event

#### Scale Pods

##### Manually

> kubectl get pod
> kubectl get deployment

Scale up to 3 pods

> kubectl scale --replicas=3 deployment.apps/shoppingclient-deployment

##### update pod Yaml file

update shoppingclient.yaml

> replicas: 2

##### create autoscale Yaml file

Check kubernetes version to confirm it allows autoscaling

> az aks show --resource-group aspnetDevOps --name aspnetDevOpsAKSCluster --query kubernetesVersion --output table

1.20.9

Create shoppingautoscale.yaml
Apply

> kubectl apply -f .\aks\

Check

> kubectl get all
> kubectl get hpa

#### Update AKS Deployment with Zero Downtime

Update Client
Remove Client docker images

> docker rmi 1b4422698bb8 -f

Rebuild image, test and tag

> cd .\Shopping\
> docker-compose -f .\docker-compose.yml -f .\docker-compose.override.yml up -d
> docker tag shoppingclient:latest aspnetdevopsshoppingacr.azurecr.io/shoppingclient:v2

Push images to registry

docker push aspnetdevopsshoppingacr.azurecr.io/shoppingclient:v2

List image in registry
az acr repository list --name aspnetdevopsshoppingacr --output table

See tag
az acr repository show-tags --name aspnetdevopsshoppingacr --repository shoppingclient --output table

Apply

> kubectl apply -f .\aks\

#### Clean All AKS and Azure Resources

az group delete --name myResourceGroup --yes --no-wait

### Pipelines

Create pipelines

### Clean All AKS and Azure Resources again

See all resources

> kubectl get all

Delete all resources

> kubectl delete -f .\aks\

Delete all Azure resources

> az group delete --name aspnetDevOps --yes --no-wait

#### kubectl clean up (users, contexts, clusters)

> kubectl config get-contexts
> kubectl config view

switch context command:

> kubectl config use-context docker-desktop
> kubectl config delete-cluster aspnetDevOpsAKSCluster
> kubectl config delete-context aspnetDevOpsAKSCluster
> kubectl config delete-user clusterUser_aspnetDevOps_aspnetDevOpsAKSCluster
