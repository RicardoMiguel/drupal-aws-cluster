# Drupal LAMP Stack on AWS Playbooks

This repository uses ansible to automate the provisioning of LAMP stack for Drupal on AWS.
It creates security groups, RDS instance and EC2 instance. All this components are eligible for AWS free-tier. 

For Drupal itself it's used this [docker image](https://hub.docker.com/r/ricardosoares/drupal-ec2/).

## Requirements

* [Ansible](https://github.com/ansible/ansible)
* [Boto](https://github.com/boto/boto3)
* [AWS account](https://aws.amazon.com/)

## Pre-Usage

You must create an AWS account and create access keys for later usage.
 
As best practice you may create an individual [IAM user](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html) and respective access keys. 
If you do so, make sure you attach the following policies, otherwise the playbooks won't work:
* AmazonRDSFullAccess
* AmazonEC2FullAccess

After acquiring access keys open your terminal and type:
 ```
 export AWS_ACCESS_KEY_ID=****
 export AWS_SECRET_ACCESS_KEY=**** 
 export AWS_REGION=*****
 ```
 
 Please note that the AWS_REGION environment variable will be used by the playbooks for creating the components in the desired region.
 You must also change this value at __inventory/ec2.ini__, variable __regions__.
 
 Ansible also needs to have an identity when facing an AWS EC2 instance. 
 For this it's used a key pair .pem file. If you don't have a key pair file please [create one](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html).
 
 After successfully creating a key pair and downloading it to your machine, type on the terminal: `ssh-add -k ~/.ssh/KEY_PAIR_NAME.pem `
 
### Reserved words
 
 If you already have an AWS account, there are some key points you must take into consideration before proceeding.
 There are some reserved words that will be used, such as:
 * drupal-rds
 * drupal-ec2-instances-group
 * drupal-rds-group
 * drupal-ec2-placeholder

If you already own components with these names, please delete those components or change the names under __group_vars/all.yml__.
Everything should work as expected.

### Configuration
There are some configuration variables you must insert in order to run the playbooks:
* __hashSaltForDrupalInstance__: Hash Salt for Drupal
* __awsVpcId__: Your AWS VPC ID
* __awsMyKeyPairName__: Key pair name to be added to EC2 instance
* __myIpForSecurityGroup__: Static IP from where you may want to connect to RDS (e.g.A server or your own machine)
* __masterUserForRds__: Username for RDS master account

Add these values in form of key=value under __sensitive-data/data.ini__.

You may also want to define your password for RDS Master account. You can do it running this command:
`
echo YOUR_RDS_PASSWORD > sensitive-data/db_password.txt
`

If you don't, ansible will generate it automatically for you.

## Usage

After all the configuration is done, you may run individual playbook at a time or run them all issuing:
```
ansible-playbook lamp-stack.yml
```

Successfully ran all the playbooks, you may connect by HTTP to your EC2 instance and note that you have to install/configure Drupal.

After configuring the initial Drupal Setup run this command to create the cluster:
```
ansible-playbook cluster.yml
```
Unfortunately the module for creating the Auto Scaling Group is not great for managing Scaling policies or notifications. Please create manually.

### Playbooks
* __aws_security_groups.yml__: Provisioning of security groups
* __aws-rds.yml__: Provisioning of RDS database instance
* __aws-rds.yml__: Provisioning of EC2 instance
* __basic.yml__: Installation of basic modules on the server for future functionalities and set timezone
* __docker.yml__: Configure docker daemon to be used
* __drupal.yml__: Installation of Apache and Drupal
* __server-drupal.yml__: Run all necessary playbooks for installing Drupal on a machine
* __lamp-stack.yml__: Run all LAMP stack related playbooks
* __aws-ami.yml__: Provisioning of default Drupal AMI
* __aws-load-balancer.yml__: Provisioning of load balancer and its security group
* __aws-launch-configuration.yml__: Provisioning of ASG Launch Configuration
* __aws-scaling-group.yml__: Provisioning of Auto Scaling Group
* __cluster.yml__: Run all Cluster related playbooks
