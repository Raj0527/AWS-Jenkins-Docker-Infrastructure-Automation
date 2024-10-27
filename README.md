# AWS Jenkins-Docker Infrastructure Automation

This project demonstrates automating the infrastructure provisioning and configuration of Jenkins and Docker servers on AWS using Terraform and Ansible. The goal is to launch and configure two EC2 instances: one for Jenkins and one for Docker.

Project Overview
1. Launch two EC2 instances in AWS: One instance will serve as the Jenkins server, and the other will host Docker.
2. Install Terraform on Jenkins server: We'll use Terraform to automate infrastructure provisioning.
3. Configure AWS CLI and Ansible: AWS CLI is used to interact with AWS services, and Ansible is used for configuration management.
4. Use Terraform to define and launch servers: We define the infrastructure as code using Terraform.
5. Install software with Ansible: Finally, we use Ansible to install necessary software packages and dependencies on both servers.

Task-1: Installing Terraform onto Anchor Server.
Once the Anchor EC2 server is up and running, SSH into the machine using MobaXterm or Putty with the username ubuntu and do the following:

sudo hostnamectl set-hostname CICDLab
bash
sudo apt update
sudo apt install wget unzip -y
wget https://releases.hashicorp.com/terraform/1.9.5/terraform_1.9.5_linux_amd64.zip
View the Terraform's Latest Versions

unzip terraform_1.9.5_linux_amd64.zip
ls
sudo mv terraform /usr/local/bin
ls
terraform -v
rm terraform_1.9.5_linux_amd64.zip
Task-2: Install Python 3, pip, AWS CLI, and Ansible
Install Python 3 and the required packages:

sudo apt-get update
sudo apt-get install python3-pip -y
sudo pip3 install awscli boto boto3 ansible
aws configure
Enter the Credentials as below. Example:
Access Key ID	   Secret Access Key

Note: If you want to create new credentials, Follow the below steps:

Go to the AWS console. On the top right corner, click on your name or AWS profile ID.
Click on Security Credentials.
Under AWS IAM Credentials, click on Create Access Key.
If you already have two active keys, you can deactivate and delete the older one so that you can create a new one, then download, and save it.
Then, Complete the aws configure step
Once configured, do a smoke test to check if your credentials are valid and get access to AWS.
You can check using any one command

aws s3 ls
(Or)

aws iam list-users
Create a host inventory file with the necessary permissions
sudo mkdir /etc/ansible && sudo touch /etc/ansible/hosts
 sudo chmod 766 /etc/ansible/hosts
Note: The above command gives read and write permissions to the owner, read and write permissions to the group, and read and write permissions to others.

Task-3: Use Terraform to launch two servers.
Create the Terraform configuration and variables files as described.

We need to create two additional servers (Docker-server and Jenkins-server, You can use t2.micro for Docker and Jenkins servers)
For Git Operations we will use the same Anchor EC2 from where we are operating now
Create the terraform directory and set up the config files in it
mkdir devops-labs && cd devops-labs
As a first step, create a key using ssh-keygen.

ssh-keygen -t rsa -b 2048 
Explanation:
-t rsa: Specifies the type of key to create, in this case, RSA.
-b 2048: Specifies the number of bits in the key, 2048 bits in this case. The larger the number of bits, the stronger the key.
Note:

This will create id_rsa and id_rsa.pub in /home/ubuntu/.ssh/
Keep the path as /home/ubuntu/.ssh/id_rsa; don't set up any passphrase, just hit the 'Enter' key for the 3 questions it asks
Now create the Terraform config files.
vi DevOpsServers.tf
Copy and paste the below code into DevOpsServers.tf

provider "aws" {
  region = var.region
}

resource "aws_key_pair" "mykeypair" {
  key_name   = var.key_name
  public_key = file(var.public_key)
}

# to create 2 EC2 instances
resource "aws_instance" "my-machine" {
  # Launch 2 servers
  for_each = toset(var.my-servers)

  ami                    = var.ami_id
  key_name               = var.key_name
  vpc_security_group_ids = [var.sg_id]
  instance_type          = var.ins_type

  # Read from the list my-servers to name each server
  tags = {
    Name = each.key
  }

  provisioner "local-exec" {
    command = <<-EOT
      echo [${each.key}] >> /etc/ansible/hosts
      echo ${self.public_ip} >> /etc/ansible/hosts
    EOT
  }
}
Now, create the variables file with all variables to be used in the main config file.

vi variables.tf
Change the following data in variables.tf File.

Edit the Allocated Region (Ex: ap-south-1) & AMI ID of same region,
Replace the same Security Group ID Created for the Anchor Server
Add your Name for KeyPair (Example: "YourName-CICDlab-KeyPair")
variable "region" {
    default = "us-east-1"
}

# Change the SG ID. You can use the same SG ID used for your CICD anchor server
# Basically the SG should open ports 22, 80, 8080, 9999, and 4243
variable  "sg_id" {
    default = "sg-06dc8863d3ed3d280" # us-east-1
}

# Choose a free tier Ubuntu AMI. You can use below. 
variable "ami_id" {
    default = "ami-0866a3c8686eaeeba" # us-east-1; Ubuntu
}

# We are only using t2.micro for this lab
variable "ins_type" {
    default = "t2.micro"
}

# Replace 'yourname' with your first name
variable key_name {
    default = "YourName-CICDlab-KeyPair"
}

variable public_key {
    default = "/home/ubuntu/.ssh/id_rsa.pub"   #Ubuntu OS
}

variable "my-servers" {
  type    = list(string)
  default = ["jenkins-server", "docker-server"]
}
Now, execute the terraform config files to launch the servers
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply -auto-approve
After the terraform code is executed, check the host's inventory file and ensure the below output.
cat /etc/ansible/hosts
It will show IP addresses of the Jenkins server and docker server as an example shown below.

[jenkins-server] 44.202.164.153
[docker-server] 34.203.249.54
(Optional Step): When you stop and start the EC2 Instances Public IP Changes. In that case, To update Jenkins & Docker Public IP addresses use the below command

sudo vi /etc/ansible/hosts 
Once Updated, Save the File.

Now SSH into Jenkins-server and check they are accessible from Anchor EC2
ssh ubuntu@<Jenkins ip address>
Set the hostname as
sudo hostnamectl set-hostname Jenkins
bash
sudo apt update
Exit only from the Jenkins Server, not the Anchor Server.

Now SSH into Docker-Server and check they are accessible from Anchor EC2
ssh ubuntu@<Docker ip address>  
Set the hostname as
sudo hostnamectl set-hostname Docker
bash
sudo apt update
Exit only from the Docker Server, not the Anchor Server.

Task-4: Use Ansible to deploy respective packages onto each of the servers
Create a directory and change to it
cd ~
mkdir ansible && cd ansible
Now, Create the playbook, which will deploy packages onto the Docker-server and Jenkins-Server.
vi DevOpsSetup.yaml
Paste the content inside the file

---

- name: Start installing Jenkins pre-requisites before installing Jenkins
  hosts: jenkins-server
  become: yes
  become_method: sudo
  gather_facts: no

  tasks:

  - name: Update apt repository with latest packages
    apt:
      update_cache: yes
      upgrade: yes

  - name: Installing jdk17 in Jenkins server
    apt:
      name: openjdk-17-jdk
      update_cache: yes
    become: yes

  - name: Installing jenkins apt repository key
    apt_key:
      url: https://pkg.jenkins.io/debian/jenkins.io-2023.key
      state: present
    become: yes

  - name: Configuring the apt repository
    apt_repository:
      repo: deb https://pkg.jenkins.io/debian binary/
      filename: /etc/apt/sources.list.d/jenkins.list
      state: present
    become: yes

  - name: Update apt-get repository with "apt-get update"
    apt:
      update_cache: yes

  - name: Finally, its time to install Jenkins
    apt: name=jenkins update_cache=yes
    become: yes

  - name: Jenkins is installed. Lets start 'Jenkins' now!
    service: name=jenkins state=started


  - name: Wait until the file /var/lib/jenkins/secrets/initialAdminPassword is present before continuing
    wait_for:
      path: /var/lib/jenkins/secrets/initialAdminPassword
  - name: You can find Jenkins admin password under 'debug'
    command: cat /var/lib/jenkins/secrets/initialAdminPassword
    register: out
  - debug: var=out.stdout_lines


- name: Start the Docker installation steps
  hosts: docker-server
  become: yes
  become_method: sudo
  gather_facts: no

  tasks:

  - name: Update 'apt' repository with latest versions of packages
    apt:
      update_cache: yes

  - name: install docker prerequisite packages
    apt:
      name: ['ca-certificates', 'curl', 'gnupg', 'lsb-release']
      update_cache: yes
      state: latest

  - name: Install the docker apt repository key
    apt_key: url=https://download.docker.com/linux/ubuntu/gpg state=present
    become: yes

  - name: Configure the apt repository
    apt_repository:
      repo: deb https://download.docker.com/linux/ubuntu bionic stable
      state: present
    become: yes

  - name: Update 'apt' repository
    apt:
      update_cache: yes

  - name: Install Docker packages
    apt:
      name: ['docker-ce', 'docker-ce-cli', 'containerd.io']
      update_cache: yes
    become: yes

  - name: Install jdk17 in Docker server. Maven needs this.
    apt:
      name: openjdk-17-jre-headless
      update_cache: yes
    become: yes

  - name: Start Docker service
    service:
      name: docker
      state: started
      enabled: yes


  - lineinfile:
       dest: /lib/systemd/system/docker.service
       regexp: '^ExecStart='
       line: 'ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:4243 -H unix:///var/run/docker.sock'


  - name: Reload systemd
    command: systemctl daemon-reload

  - name: docker restart
    service:
      name: docker
      state: restarted
...

Run the above playbook to deploy the packages
ansible-playbook DevOpsSetup.yaml
At the end of this step, the Docker-Server and Jenkins-Server will be ready for performing further Labs
Verify that the Jenkins landing page is working.
Use your respective Jenkin'sPublic IP address as shown: http://44.202.164.153:8080/
Verify that the Docker landing page is working.
Use your respective Docker's Public IP address as shown: http://34.203.249.54:4243/version
