# Kubernetes-Web-Application

In this project I deploying a Web Application in AWS Elastic Kubernetes Service (EKS).

Below are the series of steps I took in order to complete this project.


### Tools and Platforms Used:

- AWS Console
- AWS CLI
- Visual Studio
- Command Line
- Ubuntu Server
- Docker
- Git Repository
- AWS Elastic Kubernetes Service
- AWS Elastic Container Repository
- AWS EC2
- AWS Roles and Policies
- Load Balancer

### Initial Setup

To get this project going, I started by opening up the AWS console, logging in as under my root account, and creating a new IAM user with AdministratorAccess policy assigned. It is a security best practice to refrain from using the root account whenever possible, and to create users with appropiate privelleges instead.

Additionally, I enabled MFA with a virtual device for the newly created IAM user. In line with the previous security best practice, enabling MFA for all accounts is highly recommended to enhanced account security.

![mfa](https://github.com/LouisXB/Kubernetes-Web-Application/assets/115196076/7be59c75-820f-47e2-9741-bd4e76c4f8f6)

After everything with the user account was successfully set up, I signed into the AWS console with the appropiate login credentials.

### Creating an Access Key

Once logged into the console, I navigated to security credentials in the top right hand corner under my account name.
From here  I selected "Create Access Key" and downloaded it to my machine

![access key](https://github.com/LouisXB/Kubernetes-Web-Application/assets/115196076/9a49b5ef-6a9f-4800-9b48-385f27e64253)

This access key will be needed later for managing our resources from the CLI

### Creating a new policy

Next under "Policies" I created a new policy named "PortfolioProject01". Below is the template used for this policy:

```{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "autoscaling:DescribeAutoScalingGroups",
                "autoscaling:DescribeAutoScalingInstances",
                "autoscaling:DescribeLaunchConfigurations",
                "autoscaling:DescribeTags",
                "autoscaling:SetDesiredCapacity",
                "autoscaling:TerminateInstanceInAutoScalingGroup",
                "ec2:DescribeLaunchTemplateVersions"
            ],
            "Resource": "*",
            "Effect": "Allow"
        }
    ]
}
 ```

### Creating a new roles

In the IAM console, I selected "create role > AWS Service > Use Case: EKS > EKS-Cluster"
Role name: PortfolioProject01_EKSrole
Selected "create role"

![EKSrole](https://github.com/LouisXB/Kubernetes-Web-Application/assets/115196076/f9068d21-21b6-4386-9505-0ed325424fdd)


Next I created another role and selected: "create role > AWS Service > Use Case: EC2 > EC2 
Assigned the following policies:

- amazoneksworkernodepolicy
- amazonec2containerregistryreadonly
- amazonEKS_CNI_Policy
- PortfolioProject01

*Notice the last policy is the one we created earlier.

Named new role "PortfolioProject01_NodeRole"
Selected "create role"

![noderole](https://github.com/LouisXB/Kubernetes-Web-Application/assets/115196076/bfb75950-e1ef-4bea-97a8-4f06a77e470b)

### Creating EC2 key pair

After the roles were created I naviaged back to EC@ dashboard > selected region US-East-1 "N.Virginia" > click key pair > create key pair > named key pair "PortfolioProject01_KeyPair" > pem file format for use with OpenSSH > create key pair

This key pair will be used for authentication purposes later.

### Coniguring AWS credentials in CLI

Before beginning this step, AWS CLI must be installed for the correct OS

Open Commmand line > Type "aws configure" > enter the access key ID ( created and downloaded earlier) > enter secret access key > region name: US-East-1 > output format: json

The AWS CLI is now properly configured

![commandpromt1](https://github.com/LouisXB/Kubernetes-Web-Application/assets/115196076/11bb0711-7fe7-4ddd-a72c-b1537be18459)

*This command was performed on my windows desktop for demonstration purposes, the remainder of the project will be in my Ubuntu cloud enviornment

### Elastic Container Registry

In the AWS Console I navigated to Elastic Container Registry > confirmed US-East-1 region > clicked create repository > selected private visibility > named my project "portfolioproject01website"
Create Repository

![containerregistry](https://github.com/LouisXB/Kubernetes-Web-Application/assets/115196076/85e0b776-5aba-4034-bacf-7752a14fee97)


### Build Image and Run Container

In this next step I opened up a terminal and cloned a Git respitory to my home folder on my Ubuntu machine. This repository houses our nodeJS javascript "web-container-application".
This is a simple image where NodeJS is installed and the application is exposed on port 8089

![gitclonecmd2](https://github.com/LouisXB/Kubernetes-Web-Application/assets/115196076/c2a421a2-5d89-402c-97e8-56c2a838aaa0)

Next I opened Visual Studio Code > Selected "Open Folder" > chose the application folder

All files loaded into VS code

Next I went to the Docker file and opened up the VS code terminal to create our image. In the terminal typed the following: 

```
sudo docker build -f Dockerfile -t portfolioproject01website:latest .
 ```
*Builds image


```
sudo docker images
 ```
*Checks a list of images


```
sudo docker run -p 8089:8089 profolioproject01website:latest
 ```
*Runs Image


```
sudo docker ps
 ```
*Checks that container is running


```
sudo docker stop d6d5b227049a
 ```
*Stops the container. The number is the container ID number


![dockerimagebuild](https://github.com/LouisXB/Kubernetes-Web-Application/assets/115196076/126f3a91-3f57-4c94-88bf-9ba6e8215bf7)

![dockerstopandrun](https://github.com/LouisXB/Kubernetes-Web-Application/assets/115196076/87d86126-2c76-4e1e-abc2-d5bbc56b9df3)


###  Pushing container to the AWS Elastic Container Repository

Next I pushed the container to ECR which would allow me to use the image in Kubernetes service. To accomplish this I performed the following actions:

Open AWS > Elastic Container Repository > website repository > select "view push commands" 

Copy and pasted the first command in the VS Code terminal: 
```
aws ecr get-login-password --region us-east-1 |sudo docker login --username AWS --password-stdin [MY ACCOUNT ID].dkr.ecr.us-east-1.amazonaws.com
 ```

Copy and pasted the third command in the VS Code terminal: 
```
sudo docker tag portfolioproject01website:latest [MY ACCOUNT NUMBER].dkr.ecr.us-east-1.amazonaws.com/portfolioproject01website:latest
 ```

Copy and pasted the fourth command in the VS Code terminal: 
```
sudo docker push [MY ACCOUNT NUMBER].dkr.ecr.us-east-1.amazonaws.com/portfolioproject01website:latest
 ```

![pushtoregistry](https://github.com/LouisXB/Kubernetes-Web-Application/assets/115196076/125135d2-4cea-45a6-845e-3879b8b1c291)


### Creating Kubernetes Cluster

Next I needed to create my Kubernetes cluster. To do this I:

Navigated to EKS in the AWS Console and click create cluster > named cluster "portfolioproject01_cluster" > selected "PortfolioProject01_EKSrole" (created earlier) > kept default vpc configuration > selected subnets "us-east-1a" and "us-east-1b" > selected default security group > selected "public cluster endpoint access" > selected "create cluster"

Cluster successfully created


### Creating a Node Group

In the cluster under the configurations section, clicked "compute" > add node group > name node group "portfolioproject01_nodegroup" > assign the IAM node role
Configured the following Node group compute options:
- Amazon Linux 2 (AL2_x86_64) AMI type
- On-Demand capacity
- Instance type: t3.medium
- Disk size: 20GiB
- Maximum size: 4 nodes

![nodegroupconfig](https://github.com/LouisXB/Kubernetes-Web-Application/assets/115196076/592f43a7-d556-483b-9d96-f76dae94beba)


Configured network by specifying the subnets our nodes will run and and assigned key pair created earlier to node group

![nodegroupkeypair](https://github.com/LouisXB/Kubernetes-Web-Application/assets/115196076/e65e9d80-b7ea-48ea-a783-beeb0820cc9b)


### Deploying Application in Kubernetes Cluster

Opened terminal window and typed: 

```
aws eks --region us-east-1 update-kubeconfig --name portfolioproject01_cluster
 ```

In Visual Studio code opened the deployment.yaml file (See repository for code)

In terminal typed:

```
kubectl apply -f deployment.yaml
 ```
In the AWS EC2 Load balancer > copied DNS name > pasted in search engine and added :8089 to the end.
Pressed Search
Application is successfully running in our EKS cluster.
In EKS > clusters > workloads, the wesbite deployment is running in our cluster

### Scaling the Elastic Kubernetes Cluster

In VS code opened "cluster-autoscaler.yaml" (see repository for code)

In VS code terminal typed: 

```
kubectl apply -f cluster-autoscale.yaml
 ```

This deploys the auto scaler.
The permissions needed for autoscaling were already defined previous in the Policy and IAM role
In thecluster workloads in AWS Console there is now a cluster-autoscaler deployment!


Checked to see if the autoscaler is working by opening the deployment.yaml file and changing the replicas from "5" to "30"
In terminal typed: 

```
kubectl apply -f deployment.yaml
 ```

Under the cluster workload in AWS more pods have been deployed.
In the Node, instead of 2 nodes, there is now 4 nodes which is the maximum amount that we defined previously.

Scaling is properly configured for the Elastic Kubernetes Cluster!

### Cleaning up the enviornment

Deleted all resources to avoid accumulating cost.
