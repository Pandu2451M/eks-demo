
# EKS Cluster & Jenkins Setup
Step-by-step record of how the EKS cluster and the self-hosted Jenkins server were set up for this project.

## Step 1: Create EKS cluster and 
        eksctl create cluster \
        --name demo-cluster-1 \
        --region ap-south-1 \
        --node-type t2.medium \
        --nodes-min 2 \
        --nodes-max 2 \
        --zones ap-south-1a,ap-south-1b
  This provisions the EKS control plane plus a managed node group of 2 t2.medium nodes spread across two availability zones (ap-south-1a, ap-south-1b).

  ## Step 2: Launch an EC2 instance for jenkins.
  Spin up a separate EC2 instance to host Jenkins — this is the machine the CI/CD pipeline will run from.

  ### Install Java (required by Jenkins)
```bash
sudo apt install fontconfig openjdk-21-jre
```
### Install jenkins(LTS)
```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins
```
Enable and start the sevice:
```bash
sudo systemctl enable jenkins
sudo systemctl start jenkins
```
  
