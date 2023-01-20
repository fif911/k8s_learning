# Useful tips of minikube

```bash
minikube status
minikube start
minikube start --nodes 2 -p multinode-demo # ? Not sure what it exactly does

eval $(minikube -p minikube docker-env) # make local docker images accesable (have to rebuild them)

minikube node add 
minikube node list 

minikube service <service_name> # creates a tunnel from k8s cluster to host machine
```