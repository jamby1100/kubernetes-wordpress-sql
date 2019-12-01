## Create the Storage Class
```sh
kubectl create -f utilities/aws_storage_class.yml
```

## Create the PersistentVolumeClaims
```sh
kubectl create -f mysql/mysql-pvc.yml
kubectl create -f wordpress/wordpress-pvc.yml
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
```

## Cleanup

```sh
# delete rs and svc
kubectl delete rs mysql
kubectl delete svc mysql
```