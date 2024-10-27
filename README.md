# AWS Jenkins-Docker Infrastructure Automation

This project demonstrates automating the infrastructure provisioning and configuration of Jenkins and Docker servers on AWS using Terraform and Ansible. My goal is to launch and configure two EC2 instances: one for Jenkins and one for Docker.

Project Overview
1. Launch two EC2 instances in AWS: One instance will serve as the Jenkins server, and the other will host Docker.
2. Install Terraform on Jenkins server: We'll use Terraform to automate infrastructure provisioning.
3. Configure AWS CLI and Ansible: AWS CLI is used to interact with AWS services, and Ansible is used for configuration management.
4. Use Terraform to define and launch servers: We define the infrastructure as code using Terraform.
5. Install software with Ansible: Finally, we use Ansible to install necessary software packages and dependencies on both servers.

![DALL·E 2024-10-27 16 27 03 - A project architecture diagram for 'AWS Jenkins-Docker Infrastructure Automation'  The diagram should show two AWS EC2 instances_ one labeled 'Jenkins](https://github.com/user-attachments/assets/e6bf3afc-970d-4e52-986b-bca1301f059f)

Task-1: Installing Terraform onto Anchor Server.
Once the Anchor EC2 server is up and running, SSH into the machine using MobaXterm or Putty with the username ubuntu and do the following:

set Hostname

```sudo hostnamectl set-hostname CICDLab```

```bash```

```sudo apt update```

```sudo apt install wget unzip -y```

Download Terraform latest version

```wget https://releases.hashicorp.com/terraform/1.9.5/terraform_1.9.5_linux_amd64.zip```

View the Terraform's Latest Versions

```unzip terraform_1.9.5_linux_amd64.zip```

```ls```

moving terraform file to perticular location

```sudo mv terraform /usr/local/bin```

To see Terraform version

```terraform -v```

Now remove zip file
```rm terraform_1.9.5_linux_amd64.zip```

Task 2: Install Python 3, pip, AWS CLI, and Ansible
In this task, I have installed Python 3, pip, AWS CLI, and Ansible to configure and manage AWS resources.

Step-by-Step Instructions
1. Install Python 3 and pip
First, I updated the package list and installed Python 3 and pip using the following commands:

```sudo apt-get update```
```sudo apt-get install python3-pip -y```

2. Install AWS CLI, Boto, and Ansible
Next, I installed AWS CLI, Boto, and Ansible using pip3:

```sudo pip3 install awscli boto boto3 ansible```

3. Configure AWS CLI
After installing AWS CLI, I configured it by running the following command:

```aws configure```
I was then prompted to enter my AWS Access Key, Secret Access Key, default region, and output format.

Example Credentials:

```
Access Key ID [None]: ------------------
Secret Access Key [None]: ---------------
Default region name [None]: us-east-1
Default output format [None]: json
```
4. Smoke Test for AWS CLI
To verify that my credentials were valid, I ran a simple command to list my S3 buckets:

```
aws s3 ls
```
Or
```
aws iam list-users
```
This confirmed that AWS CLI was properly configured.

5. Create Ansible Host Inventory File
To configure Ansible, I created a host inventory file by running the following commands:

```
sudo mkdir /etc/ansible && sudo touch /etc/ansible/hosts
sudo chmod 766 /etc/ansible/hosts
```

I have successfully completed this task, and now I can use Ansible and AWS CLI to automate infrastructure management.

