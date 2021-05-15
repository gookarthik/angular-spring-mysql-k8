# After downloading aws-cli
Access Key ID: XXXXXXXXXXXXXXXXXXXX

Secret Access Key: XXXXXXXXXXXXXXXXXXXX

$ sudo apt update 

$ sudo apt install awscli

$ aws configure
```
AWS Access Key ID [None]: XXXXXXXXXXXXXXXXX
AWS Secret Access Key [None]: XXXXXXXXXXXXXXXXXXXX
Default region name [None]: us-east-1
Default output format [None]: json
```
# Install kubectl on ubuntu
$ curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

$ sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

$ kubectl version --client

# Install eksctl on ubuntu

$ curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

$ sudo mv /tmp/eksctl /usr/local/bin

$ eksctl version

# Create EKS Cluster using eksctl

$ eksctl create cluster --name=eksdemo1 --version 1.16 --region=us-east-1 --zones=us-east-1a,us-east-1b --without-nodegroup

$ eksctl get clusters

$ eksctl utils associate-iam-oidc-provider --region us-east-1 --cluster eksdemo1 --approve

```
eksctl create nodegroup --cluster=eksdemo1 \
                        --region=us-east-1 \
                        --name=eksdemo1-ng-private1 \
                        --node-type=t3.medium \
                        --nodes-min=2 \
                        --nodes-max=4 \
                        --node-volume-size=20 \
                        --ssh-access \
                        --ssh-public-key=kube-demo \
                        --managed \
                        --asg-access \
                        --external-dns-access \
                        --full-ecr-access \
                        --appmesh-access \
                        --alb-ingress-access \
```
kubectl get nodes -o wide

Security Group
---
 Go to 'eksdemo1'. In 'Networking', click on "Cluster security group" [sg-02881eaffdfc6890c]. Edit inbound rules to 'All Traffic'

List Service Accounts
---
$ kubectl get sa -n kube-system

Create ClusterRole, ClusterRoleBinding & ServiceAccount
---
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/master/docs/examples/rbac-role.yaml

List Service Accounts
---
$ kubectl get sa -n kube-system

Describe Service Account alb-ingress-controller 
---
$ kubectl describe sa alb-ingress-controller -n kube-system

Now get the Policy ARN of AdministratorAccess
---
arn:aws:iam::aws:policy/AdministratorAccess

Create an IAM role for the ALB Ingress Controller and attach the role to the service account
---
```
eksctl create iamserviceaccount \
    --region us-east-1 \
    --name alb-ingress-controller \
    --namespace kube-system \
    --cluster eksdemo1 \
    --attach-policy-arn arn:aws:iam::aws:policy/AdministratorAccess \
    --override-existing-serviceaccounts \
    --approve
```
# Verify using eksctl cli
 Get IAM Service Account
 --
$ eksctl  get iamserviceaccount --cluster eksdemo1

# Verify k8s Service Account
Describe Service Account alb-ingress-controller 
--
$ kubectl describe sa alb-ingress-controller -n kube-system

# Deploy ALB Ingress Controller
Deploy ALB Ingress Controller
--
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/master/docs/examples/alb-ingress-controller.yaml

Verify Deployment
---
$ kubectl get deploy -n kube-system


Edit ALB Ingress Controller Manifest
---
$ kubectl edit deployment.apps/alb-ingress-controller -n kube-system

Replaced cluster-name with our cluster-name eksdemo1
---
```
spec:
      containers:
      - args:
        - --ingress-class=alb
        - --cluster-name=eksdemo1
```
# Verify our ALB Ingress Controller is running
Verify if alb-ingress-controller pod is running
--
$ kubectl get pods -n kube-system

# Verify logs
$ kubectl logs -f $(kubectl get po -n kube-system | egrep -o 'alb-ingress-controller-[A-Za-z0-9-]+') -n kube-system

# Backend Spring-boot application deployment
First clone project from below repo, and follow README.md file for next steps

https://github.com/gookarthik/springboot-angular-kubernetes/tree/master


# Frontend Angular application deployment
First clone project from below repo, and follow README.md file for next steps

https://github.com/gookarthik/angular8-crud-demo/tree/master


# SSL
Create a SSL from Certificate Manager Service 'vvcekarthik.tk'

# Route53
Create a Hostzone called 'vvcekarthik.tk'

Create Record set for SSL
```
Record name = _bf170d3fe64ea1823d2d17dcbd54d7b4.vvcekarthik.tk
Record type = CNAME
Value = _1fe19c8347b68db4e78b9086727d5531.zzxlnyslwt.acm-validations.aws
Alias = No
TTL (seconds) = 300
Routing policy = Simple
```
```
Record name = angular.vvcekarthik.tk
Record type = A
Value = dualstack.6ad0ef5b-default-ingressus-ea9e-1111810599.us-east-1.elb.amazonaws.com.
Alias = Yes
TTL (seconds) = -
Routing policy = Simple
```

# Creating Dashboard for k8
Deploy the Kubernetes dashboard
--
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.5/aio/deploy/recommended.yaml

Create an eks-admin service account and cluster role binding
--
$ vi eks-admin-service-account.yaml
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: eks-admin
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: eks-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: eks-admin
  namespace: kube-system
```
$ kubectl apply -f eks-admin-service-account.yaml

Connect to the dashboard
--
$ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep eks-admin | awk '{print $1}')
```
ubuntu@ip-172-31-7-153:~/karthik-use$ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep eks-admin | awk '{print $1}')
Name:         eks-admin-token-kwqcb
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: eks-admin
              kubernetes.io/service-account.uid: 80485f0e-6f6a-4335-ae8d-0ccec40c9587

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6InVtWVNra3BYVmV2UDZ6SmEwMDY3d1JKSjZMMHpKUEFqSXJrMy10SDRrQ1kifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJla3MtYWRtaW4tdG9rZW4ta3dxY2IiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZWtzLWFkbWluIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiODA0ODVmMGUtNmY2YS00MzM1LWFlOGQtMGNjZWM0MGM5NTg3Iiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOmVrcy1hZG1pbiJ9.K6FANlU3HDhq8w5fcAItsoE81-OoqedUxjTF8Cs7hcCVvRjqAB4UuzrJfKjiJz_Ypcv27sCWiY9y_N5P21YN0-lb9nQwvMoooAs-y2TLkztdJTmY7QiqLmX4OjLAhESeC0upvK_4a5HJwUoxzV7HVDXRiYqS_ySGvfOmliqUB5xiVypvCgSD5xJ7U_jKnKdcRoxtg5SROLP6Wud8GVIsgFNHtfC-M2oikRhyj0aIRllEeK82Yr-JewwFWWw0N0C5Wfo8zi9w2gUdx6ZlNB-4pYvijHNuXxRqXpKqIyOekNgaIVnztg923XG3AHJAQlfZ1hncIpcA8klLF1yiSb6KKg
ubuntu@ip-172-31-7-153:~/karthik-use$
```
$ kubectl get ns

$ kubectl -n kubernetes-dashboard get svc

$ kubectl -n kubernetes-dashboard edit svc kubernetes-dashboard
 ```
 Here replace ClusterIP  to  NodePort
 ```
 $ kubectl -n kubernetes-dashboard get svc
 ```
 ubuntu@ip-172-31-7-153:~/karthik-use$ kubectl get nodes -o wide
NAME                             STATUS   ROLES    AGE   VERSION               INTERNAL-IP      EXTERNAL-IP      OS-IMAGE         KERNEL-VERSION                  CONTAINER-RUNTIME
ip-192-168-12-137.ec2.internal   Ready    <none>   28h   v1.16.15-eks-ad4801   192.168.12.137   34.200.242.167   Amazon Linux 2   4.14.231-173.361.amzn2.x86_64   docker://19.3.13
ip-192-168-61-84.ec2.internal    Ready    <none>   28h   v1.16.15-eks-ad4801   192.168.61.84    3.83.89.187      Amazon Linux 2   4.14.231-173.361.amzn2.x86_64   docker://19.3.13
ubuntu@ip-172-31-7-153:~/karthik-use$
```
```
ubuntu@ip-172-31-7-153:~/karthik-use$ kubectl -n kubernetes-dashboard get svc
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
dashboard-metrics-scraper   ClusterIP   10.100.94.72    <none>        8000/TCP        22m
kubernetes-dashboard        NodePort    10.100.54.217   <none>        443:30481/TCP   6m2s
```
# To Access Dashboard
```
https://3.83.89.187:30481/
```
Give the above token and click on sign-in. No need to open any port

# Monitoring EKS using CloudWatch Container Insigths

```
# Sample Role ARN
arn:aws:iam::180789647333:role/eksctl-eksdemo1-nodegroup-eksdemo-NodeInstanceRole-1FVWZ2H3TMQ2M

# Policy to be associated
Associate Policy: CloudWatchAgentServerPolicy
```
Deploy CloudWatch Agent and Fluentd as DaemonSets
---
```
# Template
curl -s https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/quickstart/cwagent-fluentd-quickstart.yaml | sed "s/{{cluster_name}}/<REPLACE_CLUSTER_NAME>/;s/{{region_name}}/<REPLACE-AWS_REGION>/" | kubectl apply -f -

# Replaced Cluster Name and Region
curl -s https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/quickstart/cwagent-fluentd-quickstart.yaml | sed "s/{{cluster_name}}/eksdemo1/;s/{{region_name}}/us-east-1/" | kubectl apply -f -

```
# Verify
```
# List Daemonsets
kubectl -n amazon-cloudwatch get daemonsets
```
Now Open Cloud Watch Service. CloudWatch -> Container Insights -> Performance monitoring

# Website

https://angular.vvcekarthik.tk/

# Source
https://www.youtube.com/watch?v=aPzpsfQtlKY

https://github.com/shameed1910/angular8-crud-demo

https://github.com/shameed1910/springboot-angular-kubernetes
