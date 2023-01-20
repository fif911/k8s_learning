# NodePort

```bash

kubectl apply -f simple-web.yml
# we can access a Node Port WITHIN (we have to be into cluster) a Cluster via any Node IP and NodePort 
# or ClusterIP of this service and Port that pods listen
# also NodePort allows to resolve name to IP !

kubectl get svc                                         
#NAME               TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
#hello-svc          NodePort    10.98.70.57    <none>        8080:30001/TCP   82m
#kubernetes         ClusterIP   10.96.0.1      <none>        443/TCP          4h6m
#postgres-service   ClusterIP   10.100.2.247   <none>        5432/TCP         94m

kubectl get nodes -o wide
#NAME       STATUS   ROLES           AGE    VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
#minikube   Ready    control-plane   4h9m   v1.25.3   192.168.49.2   <none>        Ubuntu 20.04.5 LTS   5.15.49-linuxkit   docker://20.10.20

# SO now if we exec INTO POD, we should be able to access: 
# 1) The pods behind NodePort via any NODE IP and 30001. As every NODE that matches selector OPENED port 30001
# and redirects every request to needed POD == curl 192.168.49.2:30001
# 2) The pods behind NodePort service via NODE PORT Service Cluster IP and 8080. 
# As pods listens to this port, and NodePort just exposes them == curl 10.98.70.57:8080
# 3) We can resolve NodePort IP just by name. So == curl hello-svc:8080




# Even if no service available - pods can talk to each other within cluster by POD ip:
# So if we exec into pod and curl another pod by ip and port it listens to - we will get response
kubectl get pods -o wide                                
#NAME                                  READY   STATUS    RESTARTS      AGE    IP           NODE      
#hello-deploy-7f86cffd47-qbhbg         1/1     Running   0             27s    172.17.0.9   minikube 
# ...

kubectl exec -it pingtest-75b7ddf679-gq62j -- bash
[root@pingtest-75b7ddf679-gq62j /]  curl  172.17.0.9:8080 # Will result into proper response
```