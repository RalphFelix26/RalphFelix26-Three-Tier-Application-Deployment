Three-Tier Web Application Deployment on AWS

This project automates the deployment of a three-tier web application using Terraform for AWS infrastructure provisioning and Jenkins for CI/CD automation.

Project Overview
The application consists of:

Frontend: A React.js web-based user interface
Backend: A Node.js REST API
Database: MongoDB for data persistence

The deployment includes:

AWS Infrastructure Provisioning with Terraform

CI/CD Pipeline using Jenkins

Network Security & Proper Environment Configurations


Techs used

AWS (EC2, VPC, Security Groups)
Terraform (Infrastructure as Code)
Jenkins (CI/CD Pipeline)
React.js (Frontend)
Node.js & Express.js (Backend)
MongoDB (Database)


Project Structure
bash

three-tier-app-deployment/
├── terraform/                
│   ├── main.tf               # AWS Provider Configuration
│   ├── variables.tf          # Terraform Variables
│   ├── outputs.tf            # Terraform Outputs
│   ├── compute.tf            # EC2 Instances
│   ├── security.tf           # Security Group Configurations
│   ├── scripts/              
│   │   ├── mongo_setup.sh    # MongoDB Setup Script
│   │   ├── backend_setup.sh  # Backend Setup Script
│   │   ├── frontend_setup.sh # Frontend Setup Script
│   └── terraform.tfvars      # Variable Values
│
├── jenkins/                  
│   └── Jenkinsfile           # CI/CD Pipeline Definition
│
└── README.md                 # Project Documentation


Prerequisites:

Terraform installed (terraform -version)
AWS CLI configured (aws configure)
Jenkins installed with necessary plugins (Pipeline, Git, AWS Credentials)
SSH Key for EC2 instances (aws ec2 create-key-pair)


Clone the Repository
bash

git clone https://github.com/your-repo/xxxxx-xxx-xxxx.git
cd three-tier-app-deployment/terraform

Configure AWS Credentials
Ensure Terraform can authenticate with AWS:

bash

export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"

Initialize and Apply Terraform
bash

terraform init
terraform apply -auto-approve

This provisions EC2 instances, security groups, and network configurations.

Configure Jenkins for CI/CD
Install Jenkins and required plugins (Pipeline, Git, AWS Credentials)
Create a new Jenkins pipeline
Use the Jenkinsfile from the repository
Trigger the build to deploy frontend and backend


Terraform Scripts

a) main.tf 

hcl

provider "aws" {
  region = "us-east-1"
}

b) variables.tf 

hcl

variable "instance_type" { default = "t2.micro" }
variable "mongo_instance_type" { default = "t2.medium" }
variable "key_name" { default = "my-key" }
variable "subnet_id" { description = "Subnet ID" }
variable "vpc_id" { description = "VPC ID" }

c) security.tf (Security Groups)

hcl

resource "aws_security_group" "mongo_sg" {
  name        = "MongoDB-SG"
  description = "Allow MongoDB access"
  vpc_id      = var.vpc_id

  ingress {
    from_port   = 27017
    to_port     = 27017
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/16"]
  }
}

d) compute.tf (EC2 Instances)

hcl

resource "aws_instance" "mongo" {
  ami           = "ami-xxxxxxxxxxxxxxxxx"
  instance_type = var.mongo_instance_type
  subnet_id     = var.subnet_id
  key_name      = var.key_name
  security_groups = [aws_security_group.mongo_sg.id]

  tags = { Name = "MongoDB-Instance" }
  user_data = file("${path.module}/scripts/mongo_setup.sh")
}

e) outputs.tf (Terraform Outputs)
hcl

output "mongo_ip" { value = aws_instance.mongo.private_ip }

CI/CD Pipeline

Jenkinsfile

groovy

pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('aws-access-key')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-key')
    }
    stages {
        stage('Terraform Apply') {
            steps {
                sh 'cd terraform && terraform apply -auto-approve'
            }
        }
        stage('Deploy Backend') {
            steps {
                sshagent(['backend-server-key']) {
                    sh 'ssh ubuntu@backend_ip "cd /home/ubuntu/backend && git pull && npm install && pm2 restart all"'
                }
            }
        }
    }
}


Setup Scripts

a) mongo_setup.sh (MongoDB Installation)
bash

#!/bin/bash
sudo apt update -y
sudo apt install -y mongodb
sudo systemctl enable mongodb
sudo systemctl start mongodb

b) backend_setup.sh (Node.js Setup)
bash

#!/bin/bash
sudo apt update -y
sudo apt install -y nodejs npm
git clone https://github.com/your-repo/backend.git /home/ubuntu/backend
cd /home/ubuntu/backend
echo "MONGO_URI=mongodb://<mongo_ip>:27017/appdb" > .env
npm install
npm start

c) frontend_setup.sh (React.js Setup)
bash

#!/bin/bash
sudo apt update -y
sudo apt install -y nginx
git clone https://github.com/your-repo/frontend.git /var/www/html/frontend
cd /var/www/html/frontend
sed -i 's/BACKEND_URL/http:\/\/<backend_ip>:3000/g' src/url.js
npm install
npm run build
sudo systemctl restart nginx
