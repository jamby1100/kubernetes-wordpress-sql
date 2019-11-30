## SQL Server

```sh
# create the replicaset and the service
kubectl create -f mysql/mysql-rs.yml
kubectl create -f mysql/mysql-svc.yml

kubectl create -f wordpress/wordpress-rs.yml
kubectl create -f wordpress/wordpress-svc.yml

kubectl get pods
kubectl get services
kubectl describe pods mysql-7xsll

# delete rs and svc
kubectl delete rs mysql
kubectl delete svc mysql
```