# Solution to the assignment problem

*******************************************************************************************

## **Phase 1: Creating basic infrastructure**

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


## **Phase 2: Creating the basic image**

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

***************************************************************************************************

## **Phase 3: Creating the repository and push images, create helm charts**

1. Created the AWS ECR repository using command below
```
aws ecr create-repository \
> --repository-name manoj-assignment-ecr \
> --image-scanning-configuration scanOnPush=true \
> --region ca-central-1
{
    "repository": {
        "repositoryUri": "386906331058.dkr.ecr.ca-central-1.amazonaws.com/manoj-assignment-ecr",
        "imageScanningConfiguration": {
            "scanOnPush": true
        },
        "encryptionConfiguration": {
            "encryptionType": "AES256"
        },
        "registryId": "386906331058",
        "imageTagMutability": "MUTABLE",
        "repositoryArn": "arn:aws:ecr:ca-central-1:386906331058:repository/manoj-assignment-ecr",
        "repositoryName": "manoj-assignment-ecr",
        "createdAt": 1627810541.0
    }
}
```

2. Tag the image and push it to the repo
```
docker tag kanban-backend:latest 386906331058.dkr.ecr.ca-central-1.amazonaws.com/manoj-assignment-ecr:latest
```

3. Pushing both the images to repo

```
docker tag kanban-ui:latest 386906331058.dkr.ecr.ca-central-1.amazonaws.com/manoj-assignment-ecr:kanban-ui
[root@ip-172-31-25-138 kanban-ui]# docker push 386906331058.dkr.ecr.ca-central-1.amazonaws.com/manoj-assignment-ecr:kanban-ui
The push refers to repository [386906331058.dkr.ecr.ca-central-1.amazonaws.com/manoj-assignment-ecr]
35e1638de2bd: Pushed
465cdbf75da4: Pushed
fbe0fc9bcf95: Layer already exists
f1b5933fe4b5: Layer already exists


kanban-ui: digest: sha256:e67670dd84275700c315dd82eb69b66f463b2f9fff920dc83149c3e19e180a7a size: 1157
[root@ip-172-31-25-138 kanban-ui]# docker tag kanban-backend:latest 386906331058.dkr.ecr.ca-central-1.amazonaws.com/manoj-assignment-ecr:kanban-backend
[root@ip-172-31-25-138 kanban-ui]# docker push 386906331058.dkr.ecr.ca-central-1.amazonaws.com/manoj-assignment-ecr:kanban-backend
The push refers to repository [386906331058.dkr.ecr.ca-central-1.amazonaws.com/manoj-assignment-ecr]
```

4. Creating the Helm chart templates for both Kanban-UI and Kanban-Backend

```
 helm create kanban-ui
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /root/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /root/.kube/config
Creating kanban-ui


 helm create kanban-backend
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /root/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /root/.kube/config
Creating kanban-backend
```

5. Modify the values.yaml and Chart.yaml for the Imagepath, Pull Policy, and tag
(Templates are uploaded under example_env folder)

6. Installation of helm charts on EKS cluster

```
 helm install kanban-ui kanban-ui/ --values kanban-ui/values.yaml
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /root/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /root/.kube/config
NAME: kanban-ui
LAST DEPLOYED: Sun Aug  1 12:10:49 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=kanban-ui,app.kubernetes.io/instance=kanban-ui" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
```

```
helm install kanban-backend kanban-backend/ --values kanban-backend/values.yaml
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /root/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /root/.kube/config
NAME: kanban-backend
LAST DEPLOYED: Sun Aug  1 12:14:41 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=kanban-backend,app.kubernetes.io/instance=kanban-backend" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
```

*******************************************************************************************************

## **Phase 4: Verfication and Troubleshooting**

1. Check if POD is deployed and running successfully

```
[root@ip-172-31-25-138 example_env]# k get pods -n default
NAME                              READY   STATUS             RESTARTS   AGE
kanban-backend-784d5f8658-59gcf   0/1     Running            0          8s
kanban-ui-6cb669995d-8dklk        0/1     CrashLoopBackOff   9          24m
```

2. Checking the logs of backend application

```
[root@ip-172-31-25-138 example_env]# k logs -f  kanban-backend-784d5f8658-59gcf -n default

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.1.6.RELEASE)

2021-08-01 12:37:18.719  INFO 1 --- [           main] c.w.medium.kanban.KanbanApplication      : Starting KanbanApplication v0.0.1-SNAPSHOT on kanban-backend-784d5f8658-59gcf with PID 1 (/app.jar started by root in /)
2021-08-01 12:37:18.724  INFO 1 --- [           main] c.w.medium.kanban.KanbanApplication      : No active profile set, falling back to default profiles: default
2021-08-01 12:37:20.862  INFO 1 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data repositories in DEFAULT mode.
2021-08-01 12:37:20.971  INFO 1 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Finished Spring Data repository scanning in 96ms. Found 2 repository interfaces.
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'liquibase' defined in class path resource [org/springframework/boot/autoconfigure/liquibase/LiquibaseAutoConfiguration$LiquibaseConfiguration.class]: Invocation of init method failed; nested exception is liquibase.exception.DatabaseException: org.postgresql.util.PSQLException: The connection attempt failed.
        at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1778) ~[spring-beans-5.1.8.RELEASE.jar!/:5.1.8.RELEASE]

```

3. As its not able to connect to database that could be an issue

4. Kanban-ui seems to be down and log analysis says nginx conf issue
(This is documented in KNOWNISSUES.md)
```
2021/08/01 13:02:45 [emerg] 1#1: host not found in upstream "kanban-app" in /etc/nginx/conf.d/default.conf:8
nginx: [emerg] host not found in upstream "kanban-app" in /etc/nginx/conf.d/default.conf:8
```

*******************************************************************************************************

## **Phase 5 : PostgresDB solution**

1. Have created a CloudFormation template for RDS DB Instance which will create te RDS and provide the endpoint

2. CloudFormation template is also created and uploaded in solution repo, below command was triggered

```
 aws cloudformation create-stack --stack-name postgresrds --template-body file://cfnpostgresrds.yaml
{
    "StackId": "arn:aws:cloudformation:ca-central-1:386906331058:stack/postgresrds/4a03c2b0-f2cd-11eb-ada3-0e75e71712c2"
}
```
2. CloudFormation template is also created and uploaded in solution repo, below command was triggered

```
 aws cloudformation create-stack --stack-name postgresrds --template-body file://cfnpostgresrds.yaml
{
    "StackId": "arn:aws:cloudformation:ca-central-1:386906331058:stack/postgresrds/4a03c2b0-f2cd-11eb-ada3-0e75e71712c2"
}
```

