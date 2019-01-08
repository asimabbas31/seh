
# Design and provision a highly available and scalable architecture that hosts a PHP application using AWS.


The purpose of the document is to design an AWS Architecture for SB PHP based application and database. The document has multiple parts to fulfill the requirements.Using the below guide you can configure the AWS nodes with Ansible Playbook, Docker, kubernetes installation, configuration and application deployment on kubernetes Cluster. Also in last part i have the added the application monitriong tool Zabbix Installation guide. 

# Architecture Desgin 

Think of AWS VPC as your virtual data center - in the cloud. Whatever you can think of doing in a data center, you do almost the same in your VPC.


![alt text](https://github.com/asimabbas31/seh/blob/asimabbas31-desgin/desgin.png)


As shown in diagram for new application we have setup the three nodes on AWS (t2.micro).Two nodes are running as kubernetes Slave cluster and one is kubernetes Master node. We are using MySql as a database storage server. On Kuberenets Slave nodes Pods are PHP application and Ngninx. In combination with PHP-FPM, Nginx is configured to send requests for .php routes to PHP-FPM to serve the page.. We’ll create a Docker image that includes our application code, and configure a pod to run containers from that image in Kubernetes. 

Furthermore to secure the environment we can use the Bastian host. Right now its not part of the my guide. 

 In very simple terms, a bastion host is the only host or computer which is allowed public network access. The other common name used for a bastion host is ‘Jump Box’. This is usually a powerful server box with high network security as this is the only host which is allowed public access. System admins can use this machine to connect to other instances in the backend infrastructure via secure authentication mechanisms.

Basic AWS Points for Better Security and Managment.

- Every region in the world has a default VPC
- Connect corporate network to the VPC using VPN
- One VPC can have only one internet gateway (igw)
- NAT gateway is an AWS software appliance, which needs to be connected to a public subnet in your VPC
- For instances in private subnet, NAT gateway or NAT (ec2) instances can be used to connect to the internet
- Security group can be considered a host firewall for each individual AWS EC2 or RDS instances, etc
- Instances on private subnet can be accessed using a jump-box , or a bastion host
- IGW are provided by AWS and are always in HA mode, spanning multiple AZ
- On Bastian host Turn on Selinux and properly configure firewall to allow only trusted access to bastion host.
- Enable system auditing by using "aureport" or other tools.
- If using multiple VPC enable VPC peering. VPC Peering allows you to communicate with other VPC in same account or with other AWS account.



For the setup please follow the pattern. 

 - Ansible Installation 

 - EC-2 Instance Creation through Ansible Playbooks

 - Kubernetes and Docker Installation through Playbooks 

 - Build Sample PHP application using Docker build 

 - Container Deployment via Kuberenets on AWS nodes 

 - Application Monitoring 



# Ansible  Installation on node: 

Install Ansible based on the OS of the machine from which you plan to execute the script. If you have a Bastian server in your infrastruce please add the ssh forwarding details using the https://blog.scottlowe.org/2015/12/24/running-ansible-through-ssh-bastion-host/


**Installation guide for UBUNTU **

Ansible Installation on Ubuntu

Prerequistie: Python , SSH, PIP, BOTO

ubuntu@ip-:~$ sudo apt-get update

ubuntu@ip-:~$ sudo apt-get install -y python-pip

ubuntu@ip-:~$ sudo apt-get install -y ansible

ubuntu@ip-:/etc/ansible$ ansible --version 

ansible 2.5.1 config file = /etc/ansible/ansible.cfg configured module search path = [u'/home/ubuntu/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules'] ansible python module location = /usr/lib/python2.7/dist-packages/ansible executable location = /usr/bin/ansible python version = 2.7.15rc1 (default, Nov 12 2018, 14:31:15) [GCC 7.3.0]

root@ip-:/etc/ansible/ansible# pip install boto Collecting boto

 Downloading https://files.pythonhosted.org/packages/23/10/c0b78c27298029e4454a472a1919bde20cb182dab1662cec7f2ca1dcc523/boto-2.49.0-py2.py3-none-any.whl (1.4MB) 100% |████████████████████████████████| 1.4MB 903kB/s Installing collected packages: boto Successfully installed boto-2.49.0

#Now Lets Move to the Ansible AWS Provision ** This tutorial explains how to manually provision an AWS EC2 virtual machine using Ansible.

The best way to get started is to install Ansible and run playbooks manually on your local machine to provision a VM.

Set AWS credentials as environment variables or you can copy it on some and use the source command set variables.

#Prepare Ansible Playbook

Ansible uses a folder structure that looks like this:
```bash
ubuntu@ip-172-31-31-214:/etc/ansible# ls
ansible.cfg asim-ssh.pem ec2_prov_playbook.retry ec2_prov_playbook.yml hosts inventory variables.yml
```
In our scenario, the important files are: ec2_prov_playbook.yml, which is the playbook that provisions an EC2 instance. variables.yml, which contains wildcard settings for the playbook.

ec2_prov_playbook.yml and ec2_term_playbook.yml scripts have some wildcards, which ansible replaces with values from variables.yml.

Please download the Ansible Playbook folder from

https://github.com/asimabbas31/seh/tree/master/ansible

In ansible.cfg, replace ${AWS_EC2_PEM_KEYPATH} with the path to the PEM key that should be used to provision the machine. Like in my file its is asim-ssh.pem
```bash
root@ip-172-31-31-214:/etc/ansible# export AWS_ACCESS_KEY_ID=XXXXXXXX
```
```bash
root@ip-172-31-31-214:/etc/ansible# export AWS_SECRET_ACCESS_KEY=XXXXXXXX
```
RUN the Playbook 

```bash
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

```
You will get information on logs or also you can verify from AWS console 

# Now our Nodes are up on AWS. Next step is to create the kubernetes Cluster on AWS.

Get the Node id from IPs from logs or from the console. 

Your cluster will include the following physical resources:

One master node
The master node (a node in Kubernetes refers to a server) is responsible for managing the state of the cluster. It runs Etcd, which stores cluster data among components that schedule workloads to worker nodes.

Two worker nodes
Worker nodes are the servers where your workloads (i.e. containerized applications and services) will run. A worker will continue to run your workload once they're assigned to it, even if the master goes down once scheduling is complete. A cluster's capacity can be increased by adding workers.

Prerequisites
- As Ansible works with SSH, it is required to exchange ssh-keys between the nodes, disable password authorization, and enable authorization by key. It is also necessary to enable root access without password.
- Python on AWS nodes 

Download the folder from 

https://github.com/asimabbas31/seh/tree/master/ansible/kube-playbooks

In hosts file update the nodes IPs and password-less username
In master-node.yml replace the user with your user "become_user: ansible"

```bash
ansible@ansible-node:~/kube-cluster$ ansible-playbook -i hosts dependencies.yml -vv
```
```bash
ansible@ansible-node:~/kube-cluster$ ansible-playbook -i hosts master-node.yml -vv
```
```bash
ansible@ansible-node:~/kube-cluster$ ansible-playbook -i hosts workers-node.yml -vv
```
Once you are done check the status your nodes

```bash
ansible@master-node:~/docker$ kubectl get nodes
NAME          STATUS   ROLES    AGE   VERSION
master-node   Ready    master   15m   v1.13.1
slave-node1   Ready    <none>   15m   v1.13.1
slave-node2   Ready    <none>   15m   v1.13.1 
```

Now lets configure the sample php based application for deployment. Kubernetes and Docker run our Nginx and PHP-FPM processes in a Kubernetes cluster. We’ll create a Docker image that includes our application code, and configure a pod to run containers from that image in Kubernetes. Running Nginx and PHP-FPM on Kubernetes. 



Application Desgin 

![alt text](https://github.com/asimabbas31/seh/blob/asimabbas31-desgin/sampleapp.PNG)


Download the kubernetes folder from https://github.com/asimabbas31/seh/tree/master/K8-deployment

Lets create the image first: 
Create you Docker Image repositary 
```bash
ansible@slave-node1:~/docker$ sudo docker run -d -p 5000:5000 --restart=always --name registry registry:2
```
Build sample application 
```bash
ansible@slave-node1:~/docker$ docker build . -t my-php-app:1.0.0
```
Tag your application to local repo 
```bash
ansible@slave-node1:~/docker$ docker tag my-php-app:1.0.0 localhost:5000/my-php-app
```
push this into local repo 
```bash
ansible@slave-node1:~/docker$ sudo docker tag my-php-app:1.0.0 localhost:5000/my-php-app
```
pull and verify image push sucessfully 
```bash
ansible@slave-node1:~/docker$ sudo docker pull localhost:5000/my-php-app
Using default tag: latest
latest: Pulling from my-php-app
Digest: sha256:97341a4e47b5b83c3be3b0b28e0545068817a5d69841f46d54f434fe9cc3bf50
Status: Image is up to date for localhost:5000/my-php-app:latest
```
```bash
ansible@slave-node1:~/docker$ sudo docker images
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
my-php-app                  1.0.0               729b6e2f33ad        About an hour ago   367MB
localhost:5000/my-php-app   latest              729b6e2f33ad        About an hour ago   367MB
```
Now we have the image in our local repo lets create the Kube pods 

```bash
ansible@master-node:~/docker$ kubectl create -f configmap.yaml 
configmap/nginx-config created
```
```bash
ansible@master-node:~/docker$ kubectl create -f web.yaml 
pod/sb-est-app created
``````bash
ansible@master-node:~/docker$ kubectl get pods
NAME                              READY   STATUS        RESTARTS   AGE
sb-est-app                        2/2     Running       0          7s
```
```bash
ansible@master-node:~/docker$ kubectl describe pod sb-est-app
Name:               sb-est-app
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               slave-node1/10.142.0.7
Start Time:         Tue, 08 Jan 2019 05:34:35 +0000
Labels:             <none>
Annotations:        <none>
Status:             Running
 ```
Key Features of APM tool: 
Key APM features:

- Monitor app performance by response times, user satisfaction, etc
- Understand performance based on application dependencies
- Identify slowest and most used web requests
- Monitor key web requests or transactions
- Identify slowest and most used SQL queries
- Monitor the performance of specific SQL queries

For Application/System/Network Monitriong there are multiple opensource tools avaible like nagios, zabbix, cacti. I am using Zabbix for the monitriong. Using the guide https://computingforgeeks.com/how-to-install-zabbix-server-4-0-on-centos-7/ . You can configure the Zabbix for your system. Also we can use the Kubernetes Dashboard for monitriong. 


![alt text](https://github.com/asimabbas31/seh/blob/asimabbas31-desgin/zabbix.png)

There are othe paid tools using that you can drill down the issues for debbuging. Like in Appdynamic it very easy to monitor the business transcation and route of the transcation with database calls. 








