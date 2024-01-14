# Kubernetes in 4 hours

## Kubernetes

[Kubernetes Cheatsheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

### Setup Dashboard

Running Kubernetes Dashboard - instructions provided by Andre Lock
[https://andrewlock.net/running-kubernetes-and-the-dashboard-with-docker-desktop/](https://andrewlock.net/running-kubernetes-and-the-dashboard-with-docker-desktop/)

To install the Kubernetes Dashboard, open terminal and run following:

- kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.5.0/aio/deploy/recommended.yaml

#### Start/Login Kubernetes Dashboard

- kubectl proxy

Login > http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

#### Setup Login User w/token

Create a user to log into Kubernetes Dashboard > https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/

Instructions for creating a sample user
https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md

- create dashboard-adminuser.yaml
- kubectl apply -f dashboard-adminuser.yaml
- create dashboard-cluster-admin-role.yaml
- kubectl apply -f dashboard-cluster-admin-role.yaml

Get the Bearer Token

- kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"

Copy token and use it to log into kubernetes-dashboard

Alternate way to get Bearer Token

- kubectl describe secret -n kube-system

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

## Class

OReilly Kubernetes sandbox

use the kubernetes dashboard to create 3 pods using image nginx

kubectl get all
kubectl get all -A
kubectl get pods -o wide
kubectl get pod firstnginx-5b8c4655fc-cv6j6 -o yaml
kubectl get all --selector app=firstnginx

kubectl describe pod firstnginx-5b8c4655fc-cv6j6

kubectl create deploy newnginx --image=nginx --replicas=3

kubectl delete pod firstnginx-5b8c4655fc-77m6f
kubectl delete replicaset.apps/firstnginx-5b8c4655fc
kubectl delete deployment deployment.apps/firstnginx

kubectl explain deploy.spec

kubectl create deploy mynginx --image=nginx --dry-run=client -o yaml > mypod.yaml

### namespaces

kubectl create ns secret
creates a namespace that other pods cannot access

### troubleshooting

kubectl deploy mydb --image=mariadb --replicas=3
kubectl geta ll --selector app=mydb
kubectl describe pod mydb-podid
kubectl logs mydb-podid

### interactive

kubectl exec -it firstnginx-podid -- sh

### set

kubectl set

Available Commands:

- env Update environment variables on a pod template
- image Update the image of a pod template
- resources Update resource requests/limits on objects with pod - - - - templates
- selector Set the selector on a resource
- serviceaccount Update the service account of a resource
- subject Update the user, group, or service account in a role binding or cluster role binding

kubectl set env deploy/mydb MARIADB_ROOT_PASSWORD=secret

### port forward

kubectl port-forward -h

> expose nginx container using 'expose'

kubcelt create deployment nginxsvc --image nginx
kubectl scale deployment nginxsvc --replicas=3
kubectl expose deployment nginxsvc --port=80
kubectl describe svc nginxsvc
kubctl get svc
kubectl get endpoints
kubectl edit nginxsvc

> edit service to add ingress exposure to allow external users to access containers

kubectl edit svc nginxsvc

### delete deployment

kubectl delete deployment firstnginx

### config map

kubcetl create cm myconf --from-file=my.conf
kubectl create cm variables --from-env-file=variables
kubectl create cm special --from-literal=VAR3=cow --from-literal=VAR4=goat
kubectl describe cm [cmname]
use --from-file to put contents of config file in the configmap
--from-env-file to define variables
--from-literal to define variables or command line

> example

kubectl create deploy mynewdb --image=mysql --replicas=3
kubectl get pods --selector app=mynewdb
kubectl create cm mynewdbvars --from-literal=MYSQL_ROOT_PASSWORD=password
kubectl describe cm mynewdbvars
kubectl set env --from=configmap/mynewdbvars deploy/mynewdb
kubectl get all --selector app=mynewdb
