# AWS Jenkins-Docker Infrastructure Automation

This project demonstrates automating the infrastructure provisioning and configuration of Jenkins and Docker servers on AWS using Terraform and Ansible. My goal is to launch and configure two EC2 instances: one for Jenkins and one for Docker.

Project Overview
1. Launch two EC2 instances in AWS: One instance will serve as the Jenkins server, and the other will host Docker.
2. Install Terraform on Jenkins server: We'll use Terraform to automate infrastructure provisioning.
3. Configure AWS CLI and Ansible: AWS CLI is used to interact with AWS services, and Ansible is used for configuration management.
4. Use Terraform to define and launch servers: We define the infrastructure as code using Terraform.
5. Install software with Ansible: Finally, we use Ansible to install necessary software packages and dependencies on both servers.


![DALL·E 2024-10-27 16 27 03 - A project architecture diagram for 'AWS Jenkins-Docker Infrastructure Automation'  The diagram should show two AWS EC2 instances_ one labeled 'Jenkins](https://github.com/user-attachments/assets/e6bf3afc-970d-4e52-986b-bca1301f059f)


## Task-1: Installing Terraform onto Anchor Server.
Once the Anchor EC2 server is up and running, SSH into the machine using MobaXterm or Putty with the username ubuntu and do the following:

set Hostname

```sudo hostnamectl set-hostname CICDLab```

```bash```

To update the list of available software packages

```sudo apt update```

```sudo apt install wget unzip -y```

Download Terraform latest version

```wget https://releases.hashicorp.com/terraform/1.9.5/terraform_1.9.5_linux_amd64.zip```


To unzip the file


```unzip terraform_1.9.5_linux_amd64.zip```

to List

```ls```

moving terraform file to particular location

```sudo mv terraform /usr/local/bin```

To see Terraform version

```terraform -v```

Now remove zip file

```rm terraform_1.9.5_linux_amd64.zip```

## Task 2: Install Python 3, pip, AWS CLI, and Ansible
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

```aws s3 ls```
Or
```aws iam list-users```
This confirmed that AWS CLI was properly configured.

5. Create Ansible Host Inventory File
To configure Ansible, I created a host inventory file by running the following commands:

```sudo mkdir /etc/ansible && sudo touch /etc/ansible/hosts
sudo chmod 766 /etc/ansible/hosts```

I have successfully completed this task, and now I can use Ansible and AWS CLI to automate infrastructure management.

## Task 3: Use Terraform to Launch Two Servers
In this task, I have used Terraform to provision and launch two EC2 instances—one for Jenkins and another for Docker.

Step-by-Step Instructions
1. Create the Directory and Set Up Configuration Files
I started by creating a directory for the Terraform configuration files:

```mkdir devops-labs && cd devops-labs```

2. Generate an SSH Key
To ensure secure SSH access to the EC2 instances, I generated an SSH key using the following command:

```
ssh-keygen -t rsa -b 2048
```

3. Create the Terraform Configuration File
I created the Terraform configuration file DevOpsServers.tf to define the two EC2 instances for Jenkins and Docker.
```vi DevOpsServers.tf```

I copied and pasted the following Terraform code:
```
provider "aws" {
  region = var.region
}

resource "aws_key_pair" "mykeypair" {
  key_name   = var.key_name
  public_key = file(var.public_key)
}

resource "aws_instance" "my-machine" {
  for_each = toset(var.my-servers)

  ami                    = var.ami_id
  key_name               = var.key_name
  vpc_security_group_ids = [var.sg_id]
  instance_type          = var.ins_type

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
```
4. Create the Variables File
I then created the variables.tf file to define the necessary variables:
```vi variables.tf```
Here is the content of the variables.tf file:

```
variable "region" {
    default = "us-east-1"
}

variable "sg_id" {
    default = "sg-06dc8863d3ed3d280" 
}

variable "ami_id" {
    default = "ami-0866a3c8686eaeeba" 
}

variable "ins_type" {
    default = "t2.micro"
}

variable "key_name" {
    default = "YourName-CICDlab-KeyPair"
}

variable "public_key" {
    default = "/home/ubuntu/.ssh/id_rsa.pub"
}

variable "my-servers" {
    type    = list(string)
    default = ["jenkins-server", "docker-server"]
}
```
I replaced the region, security group ID, and key pair name with my specific values for this project.

5. Execute the Terraform Configuration
Next, I initialized and applied the Terraform configuration:
```
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply -auto-approve
```
Terraform successfully provisioned the EC2 instances for Jenkins and Docker. After the execution, I checked the Ansible hosts inventory to ensure that the public IPs of the two servers were added:

```cat /etc/ansible/hosts```

6. Update Public IP Addresses (Optional)
Since public IPs change when EC2 instances stop and start, I updated the hosts file if necessary:
```sudo vi /etc/ansible/hosts```

7. Access Jenkins and Docker Servers
Finally, I SSH’d into both the Jenkins and Docker servers to verify they were accessible and set their hostnames:

Access Jenkins Server
```
ssh ubuntu@<Jenkins-IP-address>
sudo hostnamectl set-hostname Jenkins
sudo apt update
exit
```
Access Docker Server
```
ssh ubuntu@<Docker-IP-address>
sudo hostnamectl set-hostname Docker
sudo apt update
exit
```
I Have successfully performed this task, and Terraform has provisioned two EC2 instances—one for Jenkins and one for Docker. Now, I can proceed with configuring Jenkins and Docker.

## Task 4: Use Ansible to Deploy Respective Packages onto Each of the Servers

Step 1: Create a Directory for Ansible Playbooks
I began by creating a directory to hold my Ansible playbooks.

Create ansible directory and go at that place
```mkdir ansible && cd ansible```

Step 2: Create the Ansible Playbook
I created a YAML file named DevOpsSetup.yaml to define the tasks for setting up Jenkins and Docker.

```vi DevOpsSetup.yaml```

Content of ```DevOpsSetup.yaml```
```
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
    apt: 
      name=jenkins 
      update_cache=yes
    become: yes

  - name: Jenkins is installed. Lets start 'Jenkins' now!
    service: 
      name=jenkins 
      state=started

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
    apt_key: 
      url=https://download.docker.com/linux/ubuntu/gpg 
      state=present
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


Step 3: Run the Ansible Playbook
After saving the playbook, I executed it to deploy the packages onto the respective servers.

```ansible-playbook DevOpsSetup.yaml```

### Conclusion
By following the above steps, I successfully set up Jenkins and Docker on their respective servers using Ansible. This automation not only streamlined the installation process but also ensured consistency across environments.



