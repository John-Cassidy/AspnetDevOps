# AspnetDevOps

Deploying .Net Microservices with K8s, AKS and Azure DevOps

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
> use CatalogDb > create and switch to CatalogDb
> db.createCollection('Products')
> db.Products.insertMany([{'Name': 'Asus Laptop','Category': 'Computers', 'Summary': 'Summary', 'Description': 'Description', 'ImageFile': 'ImageFile', 'Price':1999.99 }, {'Name': 'HP Laptop','Category': 'Computers', 'Summary': 'Summary', 'Description': 'Description', 'ImageFile': 'ImageFile', 'Price':1205.99 }])
> db.Products.find({}).pretty() > get all
> db.Products.remove({}) > remove all

## Kubernetes

### Setup Dashboard

Instructions - https://andrewlock.net/running-kubernetes-and-the-dashboard-with-docker-desktop/

start Kubernetes Dashboard - kubectl proxy

Login - localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login

### Commands

kubectl
see organized commands

kubectl --help
kubectl version
kubectl cluster-info
kubectl get nodes
kubectl get pod
kubectl get services
kubectl get all
kubectl get all -- pods, services, deployments..

---

Imperative - Declarative
Imperative Commands

kubectl run [container_name] --image=[image_name]
kubectl port-forward [pod] [ports]

kubectl create [resource]
kubectl apply [resource] -- create or modify resources

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

kubectl apply -f .\nginx-deply.yaml
see deployment by running: kubectl get all:

- pod
- replicaset
- deployment

kubectl create -f
kubectl edit -f
kubctl delete -f

---

Connect to pod: kubectl exec
Historical info of pod: kubectl describe

---

Create Service on Kubernetes

nginx-service.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - portocol: TCP
      port: 80
      targetPort: 8080
```

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

section 8: create kubernetes local environment:

- Shopping.API yaml
- Shopping.Client yaml
- Mongo Db yaml
- deployment yaml file
- service yaml file
- config map
- secret definitions - storing database connection definitions
