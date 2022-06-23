# EKS-Cluster
How to build EKS cluster on AWS, in this example, I all use ap-southeast-2 (Sydney

## set AWS crendential in your computer
```
aws configure
```
or edit your credentials file, it's under ~/.aws/credentials
or use AWS SSO

remember to set the default region to ap-southeast-2

## install kubectl

notice! use kubectl 1.22, because eksctl is 1.22, the kubectl and eksctl version can't exceed -1/1 version
[https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html)

## install eksctl

[https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)

## install Helm
[https://www.eksworkshop.com/beginner/060_helm/helm_intro/install/index.html](https://www.eksworkshop.com/beginner/060_helm/helm_intro/install/index.html)

## Create EKS cluster
```
eksctl create cluster --name eks-demo-cluster --nodegroup-name linux-nodes --node-type t2.medium --nodes 2 --nodes-min 2 --nodes-max 4 --region ap-southeast-2 --zones=ap-southeast-2a,ap-southeast-2b,ap-southeast-2c
```

## Get EKS Cluster service
```
eksctl get cluster --name eks-demo-cluster --region ap-southeast-2
```

## Create IAM OIDC provider
```
eksctl utils associate-iam-oidc-provider --region ap-southeast-2 --cluster eks-demo-cluster --approve
```

## Create an IAM policy called
```
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
```

## Create a IAM role and ServiceAccount
```
eksctl create iamserviceaccount --cluster=eks-demo-cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name "AmazonEKSLoadBalancerControllerRole" --attach-policy-arn=arn:aws:iam::454377924894:policy/AWSLoadBalancerControllerIAMPolicy --approve
```

## Deploy the Helm chart
```
helm repo add eks https://aws.github.io/eks-charts
```

## Configure AWS ALB (Apllication Load Balancer) to sit infront of Ingress
```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=eks-demo-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller 
```


## Deploy Sample Application （game-2048 deployment, and ingress)
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.4.2/docs/examples/2048/2048_full.yaml
```

## Get EKS Pod data.
```
kubectl get pods --all-namespaces
```

## Verify Ingress
```
kubectl get ingress/ingress-2048 -n game-2048
```

you'll see
```
NAME           CLASS   HOSTS   ADDRESS                                                                        PORTS   AGE
ingress-2048   alb     *       k8s-game2048-ingress2-a91471f868-1855553634.ap-southeast-2.elb.amazonaws.com   80      29s
```

in browser open url: k8s-game2048-ingress2-a91471f868-1855553634.ap-southeast-2.elb.amazonaws.com

success!

<img width="670" alt="image" src="https://user-images.githubusercontent.com/4045611/175227724-6c6797ce-10c6-4520-9cee-7cfa849e9cfc.png">


## Delete EKS cluster
```
eksctl delete cluster --name eks-demo-cluster --region ap-southeast-2
```

### Clean other resources 
go to your aws dashboard

remove these in order
1. remove LB first
2. security group 
3. VPC
4. cloudformation stack
