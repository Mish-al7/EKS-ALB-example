# EKS-ALB-example
2048 game deployed using EKS, ALB as Ingress and VPC. The image of the 2048 game is already uploaded in the registry and the details is given inside the deployment manifest. 

step 1: Install and set-up AWScli and configure (aws cofigure) using access key and secret key from your AWS account.

step 2: set up kubectl(search for kubectl download and follow the steps) and eksctl in your local machine.

step 3: Once kubectl is installed, you need to configure it to work with your EKS cluster.From your AWS console or using cli with "aws eks update-kubeconfig --name your-cluster-name". verify using "kubectl get nodes" commands in bash.

step 4: create an EKS cluster from your AWS console or using AWScli using "eksctl create cluster --name demo-cluster --region us-east-1 --fargate" command. This will create clusters, take care of VPC, subnets, groups and also create fargate resource as nodes in your EKS cluster. Yes we are using fargate instead of ec2 as nodes. Fargate is a serverless resource.

step 5: Create a fargate profile: "eksctl create fargateprofile \
    --cluster demo-cluster \
    --region us-east-1 \
    --name alb-sample-app \
    --namespace game-2048"

step 6: deploy the deployment, service and Ingress : "kubectl apply -f full.yaml"

step 7: configure an OIDC-connector for applying IAM to our loadbalancer. "eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve"

step 8: Download the IAM policy. "curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json"

step 9: Create IAM policy : "aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json"

Create IAM role: "eksctl create iamserviceaccount \
  --cluster=<your-cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve"

step 10: deploy ALB controller(make sure to install helm first on your system) : "helm repo add eks https://aws.github.io/eks-charts" 

update the repo: "helm repo update eks"

install : "helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<region> \
  --set vpcId=<your-vpc-id>"

verify the deployments are running: "kubectl get deployment -n kube-system aws-load-balancer-controller"

step 11: take the address from kubectl (kubectl get ingress -n kube-system) or from loadbalancer tab in console (DNS name) and paste it in a browser and see if the 2048 game is running.
 
 