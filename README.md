## I. Infrastructure Setup

##### Architectural Diagram for Wordpress

![wordpress-architectural-diagram](https://la-jamby-catpics.s3-ap-southeast-1.amazonaws.com/wordpress-architectural-diagram.png)

##### Architectural Diagram for MySQL

![mysql-architectural-diagram](https://la-jamby-catpics.s3-ap-southeast-1.amazonaws.com/mysql-architectural-diagram.png)

## II. Setup the EKS Cluster

1. Setup your own EC2 instance. Ideally, it should have the Amazon Linux 2 AMI (so aws-cli comes pre-installed.)
2. SSH into the instance
3. Follow the cheatsheet below to create your very own EKS cluster.

```sh
# ensure aws-cli is installed
aws --version

# install aws-iam-authenticator
curl -o aws-iam-authenticator https://amazon-eks.s3-us-west-2.amazonaws.com/1.14.6/2019-08-22/bin/linux/amd64/aws-iam-authenticator
chmod +x ./aws-iam-authenticator
mkdir -p $HOME/bin && cp ./aws-iam-authenticator $HOME/bin/aws-iam-authenticator && export PATH=$HOME/bin:$PATH
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
aws-iam-authenticator help

# put in aws AKID and Secret
aws configure

# install eksctl
curl --silent --location "https://github.com/weaveworks/eksctl/releases/download/latest_release/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tm
sudo mv /tmp/eksctl /usr/local/bin
eksctl version

# install kubectl
curl -o kubectl https://amazon-eks.s3-us-west-2.amazonaws.com/1.14.6/2019-08-22/bin/linux/amd64/kubectl
chmod +x ./kubectl
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
kubectl version --short --client

# create a cluster
# - and then wait for it
eksctl create cluster \
--name prod-wordpress \
--version 1.14 \
--region ap-southeast-1 \
--nodegroup-name standard-workers \
--node-type t3.medium \
--nodes 2 \
--nodes-min 1 \
--nodes-max 4 \
--managed
```

## III. Setup the EFS File System

- Follow (this AWS doc page)[https://docs.aws.amazon.com/eks/latest/userguide/efs-csi.html] to create your EFS File system. Here are a few reminders that can help you from doing mistakes:
  - make sure the EFS File System is in the same VPC as the EKS Cluster!!
  - make sure you get the SG write.
- We need to know if your EFS File System works. 
  - Create an EC2 instance on the same VPC as your EKS cluster
  - Follow (this AWS guide)[https://docs.aws.amazon.com/efs/latest/ug/wt1-test.html] to mount your EFS File system on that EC2 instance
  - Do `sudo git clone https://github.com/jamby1100/static-kubernetes-wordpress-assets.git` to add static assets
  - Follow Section III
  - Then, visit `<<ELB URL>>/static-kubernetes-wordpress-assets/static_page.html` to see the static website
    - for me, that's `http://a6edc20da142f11ea93a802fda8e2bd4-242778073.ap-southeast-1.elb.amazonaws.com/static-kubernetes-wordpress-assets/static_page.html`

## III. Setup the Application

### 3.0 Setup this repo
```
git clone https://github.com/jamby1100/kubernetes-wordpress-sql.git
cd kubernetes-wordpress-sql

```

### 3.1 Setup Secrets

**3.1.1** Take your password and have it base64 encoded here: https://www.base64encode.org/

**3.1.2** Create a yml file for the secret `my-sql-db-secret.yml`. This file is under .gitignore so you won't see my secret.
```yml
apiVersion: v1
kind: Secret
metadata:
   name: mysql-credentials
type: Opaque
data:
  mysql_password: << REPLACE ME >>
```

**3.1.3** Create your secret
```sh

kubectl create -f mysql/my-sql-db-secret.yml

kubectl get secrets mysql-credentials
kubectl describe secrets mysql-credentials
```

### 3.1 Setup Storage

I decided to have 2 kinds of storage setups for my Kubernetes clustrer:
- MySQL: For my MySQL database, I decided to have a ReadWriteOnce EBS volume.

##### Create the Storage Class

```sh
kubectl create -f utilities/aws_storage_class.yml
kubectl create -f utilities/efs-sc.yml
```

##### Create the Volume
```sh
kubectl create -f wordpress/efs-pv.yml

kubectl get pv
```

##### Create the PersistentVolumeClaims
```sh
kubectl create -f mysql/mysql-pvc.yml
kubectl create -f wordpress/efs-pvc.yml

kubectl get pvc
```

### 3.2 Create the Replication Set

```sh
# create the replicaset
kubectl create -f mysql/mysql-rs.yml
kubectl create -f wordpress/wordpress-rs.yml

# see if it really created!
kubectl get pods
kubectl get rs
```

### 3.3 Create the Service

```sh
kubectl create -f mysql/mysql-svc.yml
kubectl create -f wordpress/wordpress-svc.yml

kubectl get services

kubectl 
```

## IV. Cleanup

```sh
kubectl delete rs mysql
kubectl delete rs wordpress

kubectl delete svc wordpress-loadbalancer
kubectl delete svc wordpress-mysql

kubectl delete pvc mysql-pvc
kubectl delete pvc efs-pvc

kubectl delete pv efs-pv
kubectl delete secrets mysql-credentials
```

## Resources

```sh
# PRIMARY REFERENCE: Kubernetes in Action by Marko Luksa (p.1-224)
https://www.manning.com/books/kubernetes-in-action

# Creating EFS File Systems for EKS 
https://docs.aws.amazon.com/eks/latest/userguide/efs-csi.html

# Mounting efs to an ec2 instance so I can add files
https://docs.aws.amazon.com/efs/latest/ug/wt1-test.html

# Doing static websites
https://www.w3schools.com/html/html_images.asp

# EKS Storage Class
https://docs.aws.amazon.com/eks/latest/userguide/storage-classes.html

# Volumes in Kubernetes
https://kubernetes.io/docs/concepts/storage/persistent-volumes/#claims-as-volumes

# Basic Wordpress on Kubernetes
https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/
```