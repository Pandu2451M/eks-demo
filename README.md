
# EKS Cluster & Jenkins Setup
Step-by-step record of how the EKS cluster and the self-hosted Jenkins server were set up for this project.

## 🚀 Step 1: Create EKS cluster
``` bash
eksctl create cluster \
--name eks-cluster \
--region ap-south-1 \
--nodegroup-name workers \
--node-type t3.medium \
--nodes 2
```
### ✅ Verify:
``` bash
kubectl get nodes
```
        
### 🔐 Grant IAM Access to the Cluster
``` bash
aws eks create-access-entry \
--cluster-name eks-cluster \
--principal-arn arn:aws:iam::010928219854:role/eksiamrole \
--type STANDARD \
--region ap-south-1
  
aws eks associate-access-policy \
--cluster-name eks-cluster \
--principal-arn arn:aws:iam::010928219854:role/eksiamrole \
--policy-arn arn:aws:eks::aws:cluster-access-policy/AmazonEKSClusterAdminPolicy \
--access-scope type=cluster \
--region ap-south-1
```
          
  This provisions the EKS control plane plus a managed node group of 2 t3.medium nodes spread across two availability zones (ap-south-1).

  ## 🖥️ Step 2: Launch an EC2 instance for jenkins.
  Spin up a separate EC2 instance to host Jenkins — this is the machine the CI/CD pipeline will run from.

  ### ☕ Install Java (required by Jenkins)
```bash
sudo apt install fontconfig openjdk-21-jre
```
### 🔧 Install Jenkins(LTS)
```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins
```
▶️ Enable and start the sevice:
```bash
sudo systemctl enable jenkins
sudo systemctl start jenkins
```
### 🌍 Open Jenkins to the Browser
In the EC2 instance's security group, add an inbound rule for port 8080.
Browse to:
```bash
http://<EC2_PUBLIC_IP>:8080
```
### 🔓 Unlock Jenkins using the initial admin password:
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
Paste the password into the setup wizard, then choose Install suggested plugins.
### 🛡️ Give the Jenkins User Sudo Access
```bash
sudo visudo
jenkins ALL=(ALL) NOPASSWD:ALL
```
Switch into the jenkins user to continue setup as that user:
```bash
sudo su - jenkins
```
### 🐳 Install Docker, Node.js, and npm
```bash
sudo apt install docker.io nodejs npm
```
### ☸️ Install kubectl
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```
Verify:
```bash
kubectl version --client
```
### 🔗 Let the jenkins User Run Docker Commands
```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```
### ☁️ Install the AWS CLI (v2)
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

## 📦 Step 3: Amazon ECR
###  🔑 login:
``` bash
aws ecr get-login-password \
| docker login \
--username AWS \
--password-stdin 010928219854.dkr.ecr.ap-south-1.amazonaws.com
```
### 🏗️ Build Image:
``` bash
docker build -t frontend .
```
### 🏷️ Tag Image:
``` bash
docker tag frontend:latest 010928219854.dkr.ecr.ap-south-1.amazonaws.com/frontend:latest
```
### ⬆️ Push Image
``` bash
docker push 010928219854.dkr.ecr.ap-south-1.amazonaws.com/frontend:latest
```

## ⛵ Step 4: Installing Helm
### 📥 Install command:
``` bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```
### ✅ Verify it installed:
``` bash
helm version
```
### 📚 Add the Helm Repo:
``` bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update
```
## ⚖️ Step 5: Installing AWS Load Balancer Controller
### 🪪 Associate OIDC Provider:
``` bash
eksctl utils associate-iam-oidc-provider \
--cluster eks-cluster \
--approve
```
### 📄 Download IAM Policy:
``` bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json
```
### 📝 Create IAM Policy:
``` bash
aws iam create-policy \
--policy-name AWSLoadBalancerControllerIAMPolicy \
--policy-document file://iam_policy.json
```
### 👤 Create IAM Service Account:
```bash
eksctl create iamserviceaccount \
--cluster=eks-cluster \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--role-name AmazonEKSLoadBalancerControllerRole \
--attach-policy-arn=arn:aws:iam::010928219854:policy/AWSLoadBalancerControllerIAMPolicy \
--approve
```
### 🛠️ Installing Controller:
``` bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
-n kube-system \
--set clusterName=eks-cluster \
--set serviceAccount.create=false \
--set serviceAccount.name=aws-load-balancer-controller
```
### ✅ Verifying:
``` bash
kubectl get deployment -n kube-system
```







  
