# AWS Jenkins-Docker Infrastructure Automation

This project demonstrates automating the infrastructure provisioning and configuration of Jenkins and Docker servers on AWS using Terraform and Ansible. The goal is to launch and configure two EC2 instances: one for Jenkins and one for Docker.

Project Overview
1. Launch two EC2 instances in AWS: One instance will serve as the Jenkins server, and the other will host Docker.
2. Install Terraform on Jenkins server: We'll use Terraform to automate infrastructure provisioning.
3. Configure AWS CLI and Ansible: AWS CLI is used to interact with AWS services, and Ansible is used for configuration management.
4. Use Terraform to define and launch servers: We define the infrastructure as code using Terraform.
5. Install software with Ansible: Finally, we use Ansible to install necessary software packages and dependencies on both servers.

Step 1: Launch EC2 Instances in AWS
Launch Jenkins and Docker Servers
Create two EC2 instances using the AWS Management Console or with Terraform (automated).

Example Terraform configuration (main.tf):

hcl
Copy code
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "jenkins_server" {
  ami           = "ami-xxxxxxxx"
  instance_type = "t2.micro"
  tags = {
    Name = "Jenkins-Server"
  }
}

resource "aws_instance" "docker_server" {
  ami           = "ami-xxxxxxxx"
  instance_type = "t2.micro"
  tags = {
    Name = "Docker-Server"
  }
}
Command to apply Terraform configuration:

bash
Copy code
terraform init
terraform apply
This will launch two EC2 instances: one for Jenkins and one for Docker.

Step 2: Install Terraform on Jenkins Server
SSH into the Jenkins server to install Terraform.

Commands:

bash
Copy code
# Update packages
sudo apt-get update

# Install Terraform
sudo apt-get install -y software-properties-common
sudo apt-add-repository --yes --update ppa:terraform/terraform
sudo apt-get install terraform
Step 3: Configure AWS CLI and Ansible
AWS CLI Configuration
Install the AWS CLI:

bash
Copy code
sudo apt-get install awscli -y
Configure the AWS CLI with your credentials:

bash
Copy code
aws configure
Provide your AWS Access Key, Secret Key, region, and output format.

Install Ansible
Install Ansible on the Jenkins server for managing the configuration of both EC2 instances:

bash
Copy code
sudo apt update
sudo apt install ansible -y
Step 4: Define Servers with Terraform
Now, use Terraform to define the instances' attributes (like instance type, region, AMI, etc.).

Create a terraform.tfvars file with the AWS region and instance details.

Example:

hcl
Copy code
aws_region = "us-east-1"
instance_type = "t2.micro"
Step 5: Launch Servers Using Terraform
Run the Terraform commands to launch the EC2 instances.

bash
Copy code
terraform init
terraform plan
terraform apply
This will automatically provision the two EC2 instances: one for Jenkins and one for Docker.

Step 6: Update Ansible Hosts File
Once the EC2 instances are running, update the Ansible hosts file with the IP addresses of the servers.

Example /etc/ansible/hosts:

ini
Copy code
[jenkins]
ec2-user@JENKINS_SERVER_IP

[docker]
ec2-user@DOCKER_SERVER_IP
Step 7: Configure Jenkins and Docker Servers
Update the hostnames of the Jenkins and Docker servers:

On Jenkins server:

bash
Copy code
sudo hostnamectl set-hostname jenkins-server
On Docker server:

bash
Copy code
sudo hostnamectl set-hostname docker-server
Step 8: Install Software with Ansible
Use Ansible to install necessary software packages on both servers. For example, installing Jenkins on one server and Docker on the other.

Jenkins Playbook (jenkins-playbook.yml):

yaml
Copy code
---
- hosts: jenkins
  become: true
  tasks:
    - name: Install Jenkins
      apt:
        name: jenkins
        state: present
Docker Playbook (docker-playbook.yml):

yaml
Copy code
---
- hosts: docker
  become: true
  tasks:
    - name: Install Docker
      apt:
        name: docker.io
        state: present
Run the playbooks:

bash
Copy code
ansible-playbook jenkins-playbook.yml
ansible-playbook docker-playbook.yml
Conclusion
This project demonstrates how to automate the infrastructure and configuration of Jenkins and Docker servers on AWS using Terraform and Ansible. By leveraging these tools, you can efficiently manage, deploy, and configure environments in a repeatable and scalable manner.


