```sh
# How to do port forwarding
kubectl port-forward kubia-manual 8888:8080
curl localhost:8888

# executing stuff inside the pod
kubectl exec kubia-tdv76 -- curl -s http://10.15.243.227

# How to get inside the container
# (1) identify the node the pod is sitting on: `kubectl get nodes -o wide`
# (2) ssh into that pod
# (3) do `docker ps`
# (4) do `docker exec -it <cid> /bin/bash`

```