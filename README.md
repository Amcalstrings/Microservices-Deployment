# Kubernetes Microservices Deployment with Jenkins Pipeline

## 📌 Project Overview
This project demonstrates the deployment of **10 microservices** in an **AWS EKS (Elastic Kubernetes Service) cluster** using a **Jenkins pipeline**. The deployment is fully automated through a **multibranch pipeline**, fetching the code from **GitHub** and deploying it to **EKS**.

## 🚀 Project Workflow
### **1️⃣ Provision EC2 Instance Using Terraform**
- A **t2.large EC2 instance** with **30GB storage** is provisioned using Terraform.
- The instance is connected to `script.sh` for automatic installation of required tools.

### **2️⃣ Install Required Tools** (via `script.sh`)
- **Docker** → Container runtime.
- **Jenkins** → CI/CD automation tool.
- **Java** → Required for Jenkins.
- **eksctl** → CLI for managing EKS clusters.
- **kubectl** → CLI for interacting with Kubernetes.
- **AWS CLI** → To manage AWS resources.

### **3️⃣ Manually Create IAM User and Assign Permissions**
The following IAM policies are granted to the IAM user:
- **AmazonEC2FullAccess**
- **AmazonEKSClusterPolicy**
- **AmazonEKSWorkerNodePolicy**
- **AWSCloudFormationFullAccess**
- **IAMFullAccess**
- **IAMUserChangePassword**

### **4️⃣ Set Up Kubernetes Cluster on AWS (EKS)**
- An **EKS cluster** is created using `eksctl`.
- Worker nodes are configured and connected to the cluster.

### **5️⃣ Configure Kubernetes Service Account & Role**
- **Create a Service Account** for authentication.
- **Create a Role** with the required permissions.
- **Bind the Role to the Service Account** for proper access control.
- **Generate a Token** using the Service Account in the desired namespace.

### **6️⃣ Deploy Microservices Using Jenkins Pipeline**
- A **multibranch pipeline** is configured in Jenkins.
- The pipeline **fetches code** from **GitHub**.
- The application is **built, containerized, and deployed** to the **EKS cluster**.

## 🛠️ Prerequisites
Ensure you have the following before starting:
- AWS account with IAM permissions for **EKS** and **EC2**.
- A **GitHub repository** containing the microservices code.
- A **Jenkins server** with Docker, `eksctl`, and `kubectl` installed.

## 🏗️ Installation & Setup
### **1️⃣ Provision EC2 Instance Using Terraform**
Create a Terraform configuration file (`main.tf`) to provision the instance:
```hcl
resource "aws_instance" "jenkins_server" {
  ami           = "<ami-id>"
  instance_type = "t2.large"
  key_name      = "<key-pair>"
  security_groups = ["<sg-id>"]
  root_block_device {
    volume_size = 30
  }
  user_data = file("script.sh")
}
```
Run the Terraform commands:
```bash
terraform init
terraform apply -auto-approve
```

### **2️⃣ Install Dependencies via `script.sh`**
Ensure `script.sh` contains:
```bash
#!/bin/bash
sudo apt update -y
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
sudo apt update -y
sudo apt install temurin-17-jdk -y
/usr/bin/java --version

#install jenkins
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y
sudo systemctl start jenkins
sudo systemctl status jenkins

#install docker
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker ubuntu
sudo usermod -aG docker jenkins
newgrp docker
sudo chmod 777 /var/run/docker.sock
sudo systemctl restart jenkins


# Install AWS CLI 
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt-get install unzip -y
unzip awscliv2.zip
sudo ./aws/install


# Install kubectl
sudo apt update
sudo apt install curl -y
curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client

# Install eksctl
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

### **3️⃣ Create an EKS Cluster**
```bash
eksctl create cluster --name my-cluster --region us-east-1 --nodegroup-name my-nodes --nodes 3
```

### **4️⃣ Configure Kubernetes Service Account & Role**
Create a service account and role with required permissions:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: my-role
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods", "services", "deployments"]
  verbs: ["get", "list", "watch", "create", "update", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: my-role-binding
  namespace: default
subjects:
- kind: ServiceAccount
  name: my-service-account
  namespace: default
roleRef:
  kind: Role
  name: my-role
  apiGroup: rbac.authorization.k8s.io
```
Create a token using the service account:
```bash
kubectl create token my-service-account --namespace default
```

### **5️⃣ Configure Jenkins Pipeline**
- Go to **Jenkins Dashboard** → **New Item** → **Multibranch Pipeline**.
- Add your **GitHub repository URL**.
- Define the `Jenkinsfile` with the deployment steps.

### **6️⃣ Deploy Microservices**
```bash
kubectl apply -f deployment.yaml
kubectl get pods -o wide
```

## 🎯 Expected Outcome
- **A Kubernetes cluster running on AWS (EKS).**
- **10 microservices successfully deployed in EKS.**
- **Jenkins pipeline automating the deployment process.**

## 📜 License
This project is open-source and available under the **MIT License**.

## 👨‍💻 Author
Created by **Caleb Amedu** 🚀



