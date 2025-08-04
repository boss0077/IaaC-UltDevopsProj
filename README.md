




Now the objective and the explanation of the project will be done later.

Below are the steps after cloning the code to run the application as planned:
Step 1:
To use IaC tf to set up the state file management we deploy S3 and dynamoDB. [for remote   backeded and state locking]
	This tf file is present in bkend folder of trfm; terraform init, plan and deploy.

Step 2:
Do the same to deploy VPC and EKS tf being present in modules.
	
Step 3:
Connect k8 cluster from CLI
	Kubectl config view  
Kubectl config current-context 	##check if cluster present in the latest name.
	##if not present use the below line to add it 
	Aws eks update-kubeconfig –region region-code –name cluster-name
	##to confirm use again - Kubectl config current-context 

Step 4:
Deploying the SA service account by 
	Kubectl apply -f serviecaccount.yaml
To check use: kubectl get sa

Step 5:
To deploy the microservices in k8s in this proj we have cumulated all the services in one deployer file.	   
Kubectl apply -f complete-deploy.yaml

Step 6:
Deploying ALB Ingress controller
	#check eksctl version
	#commands to configure IAM OIDC provider
	Export cluster_name =my-eks-cluster
oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer"  `	--output text | cut -d '/' -f 5) 
 
#Check if there is an IAM OIDC provider configured already
aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4\n

#If not, run the below command
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve

##Download IAM policy
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json


##Create IAM Policy
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json


##Create IAM Role
eksctl create iamserviceaccount \
  --cluster=<your-cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
--attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
