```bash

# Get CIRD blocks of nodes 
kubectl get nodes -o jsonpath='{.items[*].spec.podCIDR}'
# 10.244.0.0/24 10.244.1.0/24 10.244.2.0/24 10.244.3.0/24


```