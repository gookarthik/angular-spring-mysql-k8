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


# List Service Accounts
$ kubectl get sa -n kube-system

# Create ClusterRole, ClusterRoleBinding & ServiceAccount
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/master/docs/examples/rbac-role.yaml

# List Service Accounts
$ kubectl get sa -n kube-system

# Describe Service Account alb-ingress-controller 
$ kubectl describe sa alb-ingress-controller -n kube-system

# Now get the Policy ARN of AdministratorAccess
arn:aws:iam::aws:policy/AdministratorAccess

# Create an IAM role for the ALB Ingress Controller and attach the role to the service account
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
# Get IAM Service Account
$ eksctl  get iamserviceaccount --cluster eksdemo1

# Verify k8s Service Account
# Describe Service Account alb-ingress-controller 
$ kubectl describe sa alb-ingress-controller -n kube-system

# Deploy ALB Ingress Controller
# Deploy ALB Ingress Controller
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/master/docs/examples/alb-ingress-controller.yaml

# Verify Deployment
$ kubectl get deploy -n kube-system


# Edit ALB Ingress Controller Manifest
$ kubectl edit deployment.apps/alb-ingress-controller -n kube-system

# Replaced cluster-name with our cluster-name eksdemo1
```
spec:
      containers:
      - args:
        - --ingress-class=alb
        - --cluster-name=eksdemo1
```
# Verify our ALB Ingress Controller is running
# Verify if alb-ingress-controller pod is running
$ kubectl get pods -n kube-system

# Verify logs
$ kubectl logs -f $(kubectl get po -n kube-system | egrep -o 'alb-ingress-controller-[A-Za-z0-9-]+') -n kube-system

# Backend Spring-boot application deployment
First clone project from below repo, and follow README.md file for next steps

https://github.com/gookarthik/springboot-angular-kubernetes/tree/master


# Frontend Anfular application deployment
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

