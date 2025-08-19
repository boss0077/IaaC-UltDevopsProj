								IaaC - Terraform


Step 1:
To use IaC tf to set up the state file management we deploy S3 and dynamoDB. [for remote backend and state locking]
		-- present in bkend folder.
Next:
	This tf file is  in bkend folder of trfm; terraform init, plan and deploy.
Step 2:
Do the same to deploy VPC and EKS tf being present in modules. --present in modules folder 



---------------------------EOF-------------------------------




Step 3:
Connect k8 cluster from CLI
	Kubectl config view  
Kubectl config current-context 	##check if cluster present in the latest name.
	##if not present use the below line to add it 
	Aws eks update-kubeconfig –region region-code –name cluster-name

	For this proj we are using the below command:
	aws eks update-kubeconfig --region us-east-1 --name OTel-eks-cluster

	##to confirm use again - Kubectl config current-context 

Step 4:
Deploying the SA service account by 
	Kubectl apply -f serviecaccount.yaml
To check use: kubectl get sa

Step 5:
To deploy the microservices in k8s in this proj we have cumulated all the services in one deployer file.	   
Kubectl apply -f complete-deploy.yaml

##here we can access the proj after this only when we change the frontend-proxy svc to LoadBalancer.
but there is a downside to that as LB is only declarative so not much of an edits can be done to LB - done only through UI.
	also another scenario is when we have multiple external access then we need to have 10LB which is expensive.
	And there will be no support for external LB service and one without CCM (cloud control manager)

##so we will use ingress instead of LB.
	So even tho we have multiple service with external access then we can use path approach and be cost effective.
	And as a best practice many company uses routing rules where the accessing to the app with IP's are restricted.
	SO the routing rules can be created only through ingress resource. 


Step 6:
Deploying ALB Ingress controller
	#check eksctl version
	#commands to configure IAM OIDC provider
	export cluster_name=OTel-eks-cluster

oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer"  `	--output text | cut -d '/' -f 5) 

to associate it :


Host --> Ingress --> Service -->Deployment --> Pod --> Container

#Check if there is an IAM OIDC provider configured already - this help to have the IAM to pod level inside the k8s to have permission to create resources outside the k8s.
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


eksctl create iamserviceaccount \
  --cluster=OTel-eks-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
--attach-policy-arn=arn:aws:iam::771826808000:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve


Deploy ALB Controller

add helm repo 
helm repo add eks https://aws.github.io/eks-charts

install 
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \            
  -n kube-system \
  --set clusterName=OTel-eks-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=us-east-1 \
  --set vpcId=vpc-021a877d5cc8d30fe

  helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=OTel-eks-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller --set region=us-east-1 --set vpcId=vpc-021a877d5cc8d30fe


  # check if 2 pods as ingress controll is running.


  Apply ingress manifest and see if the alb is up.

  Since we dont have DNS we have to over write it in our personal lap with the IP which we get by nslookup 

  the path to put the info is - sudo vim /etc/hosts 



    - to install Gitops ie ArgoCD

  https://argo-cd.readthedocs.io/en/stable/getting_started/
  