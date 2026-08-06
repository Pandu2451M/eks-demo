# EKS Cluster & Jenkins Setup
Step-by-step record of how the EKS cluster and the self-hosted Jenkins server were set up for this project.

Step 1: Create EKS cluster and 
        eksctl create cluster \
        --name demo-cluster-1 \
        --region ap-south-1 \
        --node-type t2.medium \
        --nodes-min 2 \
        --nodes-max 2 \
        --zones ap-south-1a,ap-south-1b
  This provisions the EKS control plane plus a managed node group of 2 t2.medium nodes spread across two availability zones (ap-south-1a, ap-south-1b).

  Step 2: Launch an EC2 instance for jenkins
  Spin up a separate EC2 instance to host Jenkins — this is the machine the CI/CD pipeline will run from.

  
