# seh
Project
Design and provision a highly available and scalable architecture that hosts a PHP application using AWS.





The purpose of the document is to design an AWS Architecture for SB PHP based application and database. The document has multiple parts to fulfill the requirements. Moreover I have added the hands-on method to dockerazie the simple PHP based application which is running on Nginx and orchestration with Kubernetets. For this implementation i have used the AWS two EC-2 nodes (t2.medium). And for EC2-Nodes configuration deployment I have used the Ansible Playbooks. This document will also cover the application security and monitoring.

Part 1 = Ansible Installation 

Part 2 = EC-2 Instance Creation through Ansible Playbooks

Part 3 = Kubernetes and Docker Installation through Playbooks 

Part 4 = Build Sample PHP application using Docker build 

Part 5 = Container Deployment via Kuberenets on AWS nodes 

Part 6 = Application Security 

Part 7 = Application Monitoring 



Ansible Installation on local node: 

Install Ansible based on the OS of the machine from which you plan to execute the script.

**Installation guide **

Ansible Installation on Ubuntu

Prerequistie: Python , SSH, PIP, BOTO

ubuntu@ip-:~$ sudo apt-get update

ubuntu@ip-:~$ sudo apt-get install -y python-pip

ubuntu@ip-:~$ sudo apt-get install -y ansible

ubuntu@ip-:/etc/ansible$ ansible --version 

ansible 2.5.1 config file = /etc/ansible/ansible.cfg configured module search path = [u'/home/ubuntu/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules'] ansible python module location = /usr/lib/python2.7/dist-packages/ansible executable location = /usr/bin/ansible python version = 2.7.15rc1 (default, Nov 12 2018, 14:31:15) [GCC 7.3.0]

root@ip-:/etc/ansible/ansible# pip install boto Collecting boto

 Downloading https://files.pythonhosted.org/packages/23/10/c0b78c27298029e4454a472a1919bde20cb182dab1662cec7f2ca1dcc523/boto-2.49.0-py2.py3-none-any.whl (1.4MB) 100% |████████████████████████████████| 1.4MB 903kB/s Installing collected packages: boto Successfully installed boto-2.49.0

**Now Lets Move to the Ansible AWS Provision ** This tutorial explains how to manually provision an AWS EC2 virtual machine using Ansible.

The best way to get started is to install Ansible and run playbooks manually on your local machine to provision a VM.

Set AWS credentials as environment variables or you can copy it on some and use the source command set variables.

Prepare Ansible Playbook

Ansible uses a folder structure that looks like this:

ubuntu@ip-172-31-31-214:/etc/ansible# ls

ansible.cfg asim-ssh.pem ec2_prov_playbook.retry ec2_prov_playbook.yml hosts inventory variables.yml

In our scenario, the important files are: ec2_prov_playbook.yml, which is the playbook that provisions an EC2 instance. variables.yml, which contains wildcard settings for the playbook.

ec2_prov_playbook.yml and ec2_term_playbook.yml scripts have some wildcards, which ansible replaces with values from variables.yml.

Please download the Ansible Playbook folder from

https://github.com/asimabbas31/seh/tree/master/ansible

In ansible.cfg, replace ${AWS_EC2_PEM_KEYPATH} with the path to the PEM key that should be used to provision the machine. Like in my file its is asim-ssh.pem

root@ip-172-31-31-214:/etc/ansible# export AWS_ACCESS_KEY_ID=XXXXXXXX

root@ip-172-31-31-214:/etc/ansible# export AWS_SECRET_ACCESS_KEY=XXXXXXXX

RUN the Playbook 


root@ip-172-31-31-214:/etc/ansible# ansible-playbook -vvv ec2_prov_playbook.yml

ansible-playbook 2.5.1

  config file = /etc/ansible/ansible/ansible.cfg


creating host via 'add_host': hostname=18.236.82.237

  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
 u'hypervisor': u'xen'}) => {

    "add_host": {

        "groups": [

            "tag_Type_t2.micro"

        ],

        "host_name": "18.236.82.237",

        "host_vars": {

            "ec2_ip_address": "18.236.82.237",

            "ec2_region": "us-west-2",

            "ec2_tag_Role": "demo_machines",


            "ec2_tag_Type": "t2.micro"


You will get information on logs or also you can verify from AWS console 

Now our Nodes are up on AWS. Next step is to create the Kubernets Cluster on AWS.

