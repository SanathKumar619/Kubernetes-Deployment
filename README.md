# Kubernetes-Deployment
****3 Tier Application Deployment using Kubernetes EKS Cluster****

* Step 1: IAM Configuration - 
  1. Create a user eks-admin with AdministratorAccess.
  2. Generate Security Credentials: Access Key and Secret Access Key.

* Step 2: EC2 Setup -
  1. Launch an Ubuntu instance in your favourite region (eg. region us-west-2).
  2. SSH into the instance from your local machine.

* Step 3: Install AWS CLI v2 - 
  1. curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
  2. sudo apt install unzip
  3. unzip awscliv2.zip
  4. sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
  5. aws configure

* Step 4: Install Docker - 
  1. sudo apt-get update
  2. sudo apt install docker.io
  3. docker ps
  4. sudo chown $USER /var/run/docker.sock

* Step 5: Install kubectl - 
  1. curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
  2. chmod +x ./kubectl
  3. sudo mv ./kubectl /usr/local/bin
  4. kubectl version --short --client

* Step 6: Install eksctl - 
  1. curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
  2. sudo mv /tmp/eksctl /usr/local/bin
  3. eksctl version

* Step 7: Setup EKS Cluster - 
  1. eksctl create cluster --name three-tier-cluster --region us-west-2 --node-type t2.medium --nodes-min 2 --nodes-max 2
  2. aws eks update-kubeconfig --region us-west-2 --name three-tier-cluster
  3. kubectl get nodes

* Step 8: Run Manifests - 
  1. kubectl create namespace workshop
  2. kubectl config set-context --current --namespace workshop
  3. kubectl apply -f .
  4. kubectl delete -f .

* Step 9: Install AWS Load Balancer - 
  1. curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
  2. aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
  3. eksctl utils associate-iam-oidc-provider --region=us-west-2 --cluster=three-tier-cluster --approve
  4. eksctl create iamserviceaccount --cluster=three-tier-cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-        
  arn=arn:aws:iam::__ACCOUNTID__:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=us-west-2

* Step 10: Deploy AWS Load Balancer Controller - 
  1. sudo snap install helm --classic
  2. helm repo add eks https://aws.github.io/eks-charts
  3. helm repo update eks
  4. helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=my-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
  5. kubectl get deployment -n kube-system aws-load-balancer-controller
  6. kubectl apply -f full_stack_lb.yaml

* Cleanup - 
To delete the EKS cluster:
  1. eksctl delete cluster --name three-tier-cluster --region us-west-2

**Note : For any queries or issues, please open an issue in the repository.**


