# USING TERRAFORM IAC TOOL TO AUTOMATE AWS CLOUD SOLUTION FOR 2 COMPANY WEBSITES (INTRODUCING TERRAFORM CLOUD, ANSIBLE & PACKER) - CONTINUATION.

In [Project-18](https://github.com/Olaminiyi/Project-18), I migrated the terraform.tfstate file to S3 bucket for easy collaboration amongst DevOps team mates in an organisation. I will be introducing [Terraform Cloud](https://developer.hashicorp.com/terraform/cloud-docs) to further automate the process.

Clone this repo to your working directory, it contains the AMI scripts that we are going to build with packer [AMI](https://github.com/Olaminiyi/PBL-project-19/blob/main/AMI/nginx.pkr.hcl) 

This documentation provides a step-by-step guide on using Terraform Cloud, Ansible, and Packer together to provision infrastructure in AWS. Terraform is used for defining and managing infrastructure as code, Ansible for configuring servers, and Packer for building machine images. By combining these tools, you can automate the provisioning process and ensure consistent, reproducible infrastructure deployments.

**BENEFITS:**

[Terraform Cloud](https://developer.hashicorp.com/terraform/cloud-docs) is an application that helps teams use Terraform together. It manages Terraform runs in a consistent and reliable environment, and includes easy access to shared state and secret data, access controls for approving changes to infrastructure, a private registry for sharing Terraform modules, detailed policy controls for governing the contents of Terraform configurations and more. Teams can connect Terraform to version control, share variables, run Terraform in a stable remote environment, and securely store remote state.

Terraform Cloud executes Terraform commands on disposable virtual machines, this remote execution is also called [remote operations](https://developer.hashicorp.com/terraform/cloud-docs/run/remote-operations).

Instead of running the Terraform codes in [Project-18](https://github.com/Olaminiyi/Project-18) from a command line, rather it is being executed from [Terraform Cloud](https://developer.hashicorp.com/terraform/cloud-docs) console. The **AMI** is built with **PACKER** while **ANSIBLE** is used to configure the infrastructure after its been provisioned with the Terraform script.

**TASKS**

- Build AMIs using Packer.
- Update the terraform script with the AMI IDs generated from packer build.
- Create terraform cloud and backend
- Run terraform script
- Update Ansible script with values from Terraform output
    - RDS endpoints for wordpress and tooling
    - Database name, password and username for wordpress and tooling
    - Access point ID for wordpress and tooling
    - Internal Load balancer DNS for Nginx reverse proxy
- Run the Ansible script
- Access the website through the browser.

**PREREQUISITE**

- Install **ANSIBLE**
- Install [PACKER](https://developer.hashicorp.com/packer/install?product_intent=packer).
- Configure [AWS CLI](https://www.youtube.com/watch?v=u0JyzUGzvJA).

> [!NOTE] 
> We create a repository from terraform files we created in project 17 additional to this folder, we will include Ansible module and AMI module. The AMI module contains packer configuration for creating our AMIs

### Create a Terraform Cloud account

Let us explore how we can migrate our codes to Terraform Cloud and manage our AWS infrastructure from there:
- Create a Terraform Cloud account.

![alt text](images/19.1.png)

**Create an organization**
- Select "Start with a blank state", choose a name for your organization and create it.
            
![alt text](images/19.2.png)

![alt text](images/19.3.png)
        
**Configure a workspace on the project and workspaces**
- click on create a workspace
- We will use version control workflow as the most common and recommended way to run Terraform commands triggered from our git repository.
- Create a new repository in your GitLab and call it PBL_project-19, push your Terraform codes developed in the previous projects to the repository.
        
![alt text](images/19.4.png)
     
- click on version control workflow
![alt text](images/19.5.png)

- under connect to VCS choose GitLab (use can use Github if you want)

![alt text](images/19.6.png)

**setup prodiver**
- click on the register a new OAuth Application link: it will take you to the VSC you chose (Github,GitLab or any)

![alt text](images/19.7.png)
          
- enter the corresponding details as shown on the page and click Next
           
 ![alt text](images/19.8.png)
       
- click continue

![alt text](images/19.9.png)

- under set up provider 
- Enter the details in the "Application: Terraform cloud(Olami)" page into the corresponding field
   
![alt text](images/19.10.png)
    
![alt text](images/19.11.png)
    
- > click Next
        
- skip set up ssh keypair
        
![alt text](images/19.12.png)

- under chose repository : chose PBL-project-19

![alt text](images/19.13.png)
        
- Give the workspace a name and desription
         
![alt text](images/19.14.png)
        
- under VCS trigger : make sure "always trigger run" and "automatic speculative plan" are selected
        
![alt text](images/19.15.png)

- > click Next
  
- Configuration upload success

 ![alt text](images/19.16.png)

![alt text](images/19.17.png)

### Configure variables
**Terraform Cloud** supports two types of variables: `environment variables` and `Terraform variables`. Either type can be marked as `sensitive`, which prevents them from being displayed in the Terraform Cloud web UI and makes them `write-only`

- click on variables on the side bar menu 

![alt text](images/19.18.png)

Click add variable : it shows Terraform variable which makes use of the terraform auto.tfvars; and Environment variable which makes use of secret and access keys. We can load our `terraform auto.tfvars` in the terraform variable but it can pick it automaticall from our project if we named the file(terraform.auto.tfvars)

Set two environment variables: `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`, set the values that you created for terraform

![alt text](images/19.21.png)

![alt text](images/19.22.png)


### lets start building images
- go to the directory you have the packer file i.e PBL 19
- packer init .
```
packer init .
``` 
> [!NOTE]
> If we have only 1 packer file in the folder, if it's more than it will throw error of duplicate. you can leave it and go to the next step.
- we want to build bastion first
```  
packer build .\bastion.pkr.hcl
```
- it was giving this error

![alt text](images/19.19.png)

> [!NOTE] 
> I resolved it by running this command : packer plugins install github.com/hashicorp/amazon
> i encountered another error
> ![alt text](images/19.23.png)
> i resolved it by changing the RHEL to RHEL-8.6.0_HVM-20220503-x86_64-2-Hourly2-GP2
> i encountered another error that the subnet was not specified
> i change the region from eu-west-2 to us-west-2
> ![alt text](images/19.24.png)

- packer build for nginx

![alt text](images/19.26.png)

- packer build for web

![alt text](images/19.25.png)

- packer build for ubuntu
- i encountered this error when building the ubuntu AMI
![alt text](images/19.27.png)
> [!NOTE]
> I was able to resolve it by changing the ami to this: 
> "ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-20240228"
>  owners      = ["099720109477"]
>   ![alt text](images/19.28.png)

All the AMIs after created successfully

![alt text](images/19.29.png)

### The next step is to copy the ami id and put them accordingly in our terrafor.auto.tvars

- go to the aws console > click on AMI > select each AMi and copy the AMI id
- go to the terraform.auto.tvars and paste the copied AMIs

![alt text](images/19.30.png)

![alt text](images/19.31.png)

Commit the changes and push it to the repo

> [!WARNING] 
> I encountered this errors
> ![alt text](images/error2.png)
> The error was as a result of case sensitivity of the terraform cloud for the AWS access ID and Key. I deleted it and added it again in captital letter

**The next step is to check the plan carry out plan and apply just like using terraform on our local machine**
- go to to terraform cloud workspace of your project
- from the side menu, click on RUN
- if their is no any "run" going on
- click on New run > Start

![alt text](images/19.33.png)

The "run" created a successful plan

![alt text](images/19.34.png)

scrow down and click on "confirm & apply"

![alt text](images/19.35.png)

> [!WARNING]
> I encountered this error

>![alt text](images/error3.png)
>![alt text](images/error4.png)

I checked the IAM console on the AWS, i checked under the roles and found that "ec2-instance-role" exist. This is as a result of my previous projects bearing the same name
> [!NOTE]
> I deleted it 

> ![alt text](images/19.36.png)

> [!IMPORTANT]
> I created another run and apply to create the remaining resources and ran into another error
> ![alt text](images/error5.png)
> I checked under the IAM console but it was not showing. I learnt that this type can persist on a system without showing under the IAM role
> I used this command to check for the list of profiles on my AWS account
```
aws iam list-instance-profiles
```
> It shows the profile exist
> deleted it with this command 
```
aws iam delete-instance-profile --instance-profile-name aws_instance_profile_test  
```   
> ![alt text](images/19.37.png)

**I created another run and apply and it was succesful**

![alt text](images/19.38.png)

![alt text](images/19.39.png)

**check the resources if they are created successfully on the AWS**

![alt text](images/19.40.png)

![alt text](images/19.41.png)

![alt text](images/19.42.png)

![alt text](images/19.43.png)

checking our target gropus, i expect them to fail health check because they have not been configured the instances like: installing apache, fixing the path for the health check.

![alt text](images/19.44.png)

![alt text](images/19.45.png)

There are two ways to do it, we can either `deregisterd` the target group or do it in the `terraform code`.The first to do is to remove those `instance` as `listener` to the `load balancer`. This is because the `target group` as been designed as a `listener` to the `load balancer` and as a result of it the `load balancer` will `forward traffick` to them.
So we are going to comment them out first from our code

![alt text](images/19.46.png)

![alt text](images/19.47.png)

![alt text](images/19.47.png)

![alt text](images/19.48.png)

Secondly, we will go into the `autoscaling group`, we know the **autoscalling group are attach to the load balancer**. so that the autoscaling group will not spin up another EC2 instance, so we are going to comment out the autoscalling group attachmnet for (nginx, wordpress and tooling)

![alt text](images/19.49.png)

![alt text](images/19.50.png)

![alt text](images/19.51.png)

Commit the changes and push to git

![alt text](images/19.52.png)

![alt text](images/19.53.png)

### The next step is to update the ansible script with values from terraform output
- RDS endpoints for wordpress and tooling
- Database name, password and username for tooling and wordpress
- Access points ID for tooling and wordpress
- internal load balancer DNS for nginx reverse proxy

To achieve the step listed above, we login to our bastion host, clone done the ansible script and make our changes.
- add the pem key to your terminal

![alt text](images/19.54.png)

- go to instances on AWS console and copy the public ip of the baston
- add it to the remote ssh configuration on the vscode

![alt text](images/19.55.png)
    
- ssh to the bastion server
- to confirm that our forward agent works ; open a terminal and type this command
```
ssh-add -l
```
![alt text](images/19.56.png)

By default because we have added our `keypair` to the `keychain`, the `ssh agent` bring it into the server but does not store it on the server.

From our AMI we installed AWS CLI, let check if it was installed with this command
```
aws
```
It was installed

![alt text](images/19.57.png)

Clone ansible folder from gitlab
```  
git clone https://gitlab.com/olaiyaolaminiyi/ansible-deploy-pbl-19.git
```
`Ansible` is going to need to connect to our `AWS account` to pull the `private ip addresses` of the instances that was we used the [dynamic inventory](https://docs.ansible.com/ansible/latest/collections/amazon/aws/docsite/aws_ec2_guide.html) inside the `aws_ec2.yml`.

We need to give ansible our secret and access key also. so we do aws configure also on our bastion and set the `access` and `secret key`.
```
aws configure
```

![alt text](images/19.58.png)

let's test it by checking for the s3 on the AWS

![alt text](images/19.59.png) 

### copy the rds endpoint for wordpress and tooling
- go to the RDS on the AWS console
- click on databases > click on terraform created database
- copy the RDS endpoint

![alt text](images/19.60.png)

- go to the ansible file > roles > tooling > tasks > setup-db.yml
- update the terraform endpoint in the creating database task and Input tooling credentials task with: 
`terraform-20240317221301736400000009.cr2aami6gg0r.us-west-2.rds.amazonaws.com`

![alt text](images/19.63.png)

- go to the ansible file > roles > wordpress > tasks > setup-db.yml
- update the terraform endpoint in the creating database task and Input wordpress credentials task with: 
`terraform-20240317221301736400000009.cr2aami6gg0r.us-west-2.rds.amazonaws.com`

![alt text](images/19.64.png)

### Copy the DNS name for the internal load balancer
- go to load balancer > select the ialb(internal load balance)
- copy the DNS name

![alt text](images/19.61.png)

- go to the ansible file > roles > nginx > template > nginx.confi.j2
- update the dns name for the load balancer

![alt text](images/19.62.png)

### The next step is to update file system ID and Access points ID for tooling and wordpress

- go to EFS on AWS console > access point
- click on the wordpress > click attach 

![alt text](images/19.65.png)

- copy the fsap part first

![alt text](images/19.66.png)

- go to the ansible file > roles > tooling > tasks > main.yml 
- paste it to replace `fsap` part for `opts`

![alt text](images/19.67.png)   

- Copy the `fs` part and replace it with the `src` part

![alt text](images/19.68.png)

- Go to the ansible file > roles > wordpress > tasks > main.yml 

![alt text](images/19.69.png)

- We do the same thing for the wordpress

![alt text](images/19.70.png)

![alt text](images/19.71.png)

![alt text](images/19.72.png)

![alt text](images/19.73.png)

![alt text](images/19.74.png)

### Install some required application on bastion
```
sudo dnf install python3-devel
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
sudo python3.11 get-pip.py
pip install botocore boto3
```

### update the ansible.cfg roles path

- open the ansible.cfg
- copy the path of the role and update the roles path
- then copy and export the Ansible.cfg path for ansible to look at the path when running

![alt text](images/19.76.png)

```
export ANSIBLE_CONFIG=/home/ec2-user/ansible-deploy-pbl-19/ansible.cfg
```
![alt text](images/19.77.png)

### The next step is to run the ansible playbook to configure the instances

We need to run the ansible inventory first to check if the dynamic inventory will be able to fetch our private IP addresses.
```
ansible-inventory -i inventory/aws_ec2.yml --graph
```
![alt text](images/19.78.png)


We can run our ansible playbook now
```
ansible-playbook -i inventory/aws_ec2.yml playbooks/site.yml
```
![alt text](images/19.94.png)
![alt text](images/19.95.png)


**Check if the target groups are healthy**

![alt text](images/19.112.png)

![alt text](images/19.113.png)

![alt text](images/19.114.png)

**Check the Listeners**

![alt text](images/19.115.png)

![alt text](images/19.116.png)

![alt text](images/19.117.png)


ssh into the nginx server from our bastion server to check if it was properly configured
- check for the nginx status
```
sudo vi /etc/nginx/nginx.conf
```
![alt text](images/19.98.png)

ssh into the tooling server from our bastion server to check if it was properly configured
- check if it was successfull mounted
```
df -h
```
![alt text](images/19.99.png)

Check if server is running
``` 
sudo systemctl status httpd 
```
![alt text](images/19.100.png)

change directory to html
```
cd /var/www/html
ls
```

![alt text](images/19.101.png)

Check if the website is running locally
```
curl localhost
```
![alt text](images/19.102.png)

ssh into the wordpress server from our bastion server to check if it was properly configured
- check if it was successfull mounted
```
df -h
```
![alt text](images/19.103.png)

check if server is running
```
sudo systemctl status httpd 
```
![alt text](images/19.104.png)

Change directory to html
```
cd /var/www/html
ls
```
![alt text](images/19.105.png)

check if the website is running locally
```
curl -v localhost
```
![alt text](images/19.106.png)
![alt text](images/19.107.png)

> [!NOTE]
> There is an error with the database connection for wordpress
> configure it
```  
sudo vi wp-config.php
```    
> ansible did not input the database name, username and password and hostname properly.

>![alt text](images/19.108.png)

> **hostname is our terraform endpoint**.

> ![alt text](images/19.109.png)

The Wordpress is connected to the databse now. The website is available locally now

![alt text](<images/Screenshot 2024-03-23 at 18.51.01.png>)

We then go to the terraform script and uncomment the `auto scaling attachment` in the `asg-bastion-nginx.tf` and `asg-tooling-wordpress.tf` and also the `listeners` in the `alb.tf`.

Push the updated terrform code to github. Terraform cloud picks up the changes, run a plan and apply.

![alt text](images/19.110.png)

![alt text](images/19.111.png)

- Go to Route53
- Under Records, copy hostnames for both tooling and wordpress respectively
- paste the hostnames on the browser

![alt text](<images/Screenshot 2024-03-23 at 19.26.52.png>)

![alt text](<images/Screenshot 2024-03-23 at 19.27.16.png>)

![alt text](<images/Screenshot 2024-03-23 at 19.27.48.png>)

![alt text](<images/Screenshot 2024-03-23 at 19.28.18.png>)











aws ec2 describe-availability-zones --region $REGION 


