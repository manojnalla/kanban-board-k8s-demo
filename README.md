#Solution to the assignment problem

*******************************************************************************************

##**Phase 1: Creating basic infrastructure**

1. Created Amazon EKS cluster for deployment testing and automation

2. Create nodegroup with 3 nodes

3. Updated local kubectl with the cluster kubeconfig using the command

  aws eks --region ca-central-1 update-kubeconfig --name eks-cluster

4. Connected to the cluster and verified

```
kubectl get nodes
NAME                                             STATUS   ROLES    AGE   VERSION
ip-172-31-17-103.ca-central-1.compute.internal   Ready    <none>   12m   v1.20.4-eks-6b7464
ip-172-31-2-105.ca-central-1.compute.internal    Ready    <none>   12m   v1.20.4-eks-6b7464
ip-172-31-37-92.ca-central-1.compute.internal    Ready    <none>   13m   v1.20.4-eks-6b7464
```

5. Installed all basic softwares on my centralized EC2 instance like git, docker, kubectl and also configured AWS CLI using credentials.

************************************************************************************************


##**Phase 2: Creating the basic image**

1. docker build -t docker-ui .

2. docker build -t kanban-backend .

3. Verified if the images are created correctly 
```
docker images
REPOSITORY       TAG                IMAGE ID       CREATED              SIZE
kanban-backend   latest             4141d5e09180   About a minute ago   151MB
<none>           <none>             11a1893aaa8a   About a minute ago   432MB
docker-ui        latest             1b9bf6c1b768   3 minutes ago        22.3MB
<none>           <none>             7aac2cfa538e   3 minutes ago        457MB
maven            3.6.1-jdk-8-slim   96a712cf9211   23 months ago        301MB
node             12.7-alpine        d97a436daee9   2 years ago          79.3MB
nginx            1.17.1-alpine      ea1193fd3dde   2 years ago          20.6MB
openjdk          8-alpine           a3562aa0b991   2 years ago          105MB
```

