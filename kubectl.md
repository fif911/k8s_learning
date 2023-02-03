# Useful commands in kubectl

Refer to [original guide](https://kubernetes.io/docs/reference/kubectl/cheatsheet/) by kubernetes developers.

```bash

# Get CIRD blocks of nodes 
kubectl get nodes -o jsonpath='{.items[*].spec.podCIDR}'
# 10.244.0.0/24 10.244.1.0/24 10.244.2.0/24 10.244.3.0/24

kubectl get all
kubectl delete -f simple-web.yml 
kubectl apply -f simple-web.yml 

kubectl get configmap <configmap-name> 
kubectl describe configmap <configmap-name> 

kubectl exec -it <pod_id> -- bash # execute command in container (interactive)
psql -h localhost -U user --password app
kubectl exec <pod_id> -- printenv # get env variables of the pod 


kubectl top node/pod <node>/<pod>
```

## Exam notes

### Quick creation of services & deployments

```bash
kubectl create deployment hello-minikube1 --image=kicbase/echo-server:1.0

kubectl edit deploy {name} # to get generated yaml

kubectl expose deployment hello-minikube1 --type=LoadBalancer --port=8080

kubectl edit svc {name} # to get generated yaml
```