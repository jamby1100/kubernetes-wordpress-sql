## Create the Storage Class
```sh
kubectl create -f utilities/aws_storage_class.yml
```

## Create the PersistentVolumeClaims
```sh
kubectl create -f mysql/mysql-pvc.yml

kubectl get pvc
```

## Create the Replication Set

```sh
# create the replicaset
kubectl create -f mysql/mysql-rs.yml
kubectl create -f wordpress/wordpress-rs.yml

# see if it really created!
kubectl get pods
kubectl get rs
```

## Create the Service

```sh
kubectl create -f mysql/mysql-svc.yml
kubectl create -f wordpress/wordpress-svc.yml

kubectl get services

kubectl exec kubia-tdv76 -- curl -s http://10.100.194.174
```

## Cleanup

```sh
kubectl delete pvc mysql-pvc
kubectl delete pvc wordpress-pvc

kubectl delete rs mysql
kubectl delete rs wordpress-mysql
kubectl delete rs wordpress

kubectl delete svc wordpress-loadbalancer
kubectl delete svc mysql
```