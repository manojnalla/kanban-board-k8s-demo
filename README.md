Solution to the assignment problem

Phase 1: Creating basic infrastructure

1. Created Amazon EKS cluster for deployment testing and automation

2. Create nodegroup with 3 nodes

3. Updated local kubectl with the cluster kubeconfig using the command

  aws eks --region ca-central-1 update-kubeconfig --name eks-cluster

4. Connected to the cluster and verified

kubectl get nodes
NAME                                             STATUS   ROLES    AGE   VERSION
ip-172-31-17-103.ca-central-1.compute.internal   Ready    <none>   12m   v1.20.4-eks-6b7464
ip-172-31-2-105.ca-central-1.compute.internal    Ready    <none>   12m   v1.20.4-eks-6b7464
ip-172-31-37-92.ca-central-1.compute.internal    Ready    <none>   13m   v1.20.4-eks-6b7464

5. Installed all basic softwares on my centralized EC2 instance like git, docker, kubectl and also configured AWS CLI using credentials.


Phase 2: Creating the basic image
