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
