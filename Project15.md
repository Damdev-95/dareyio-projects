
## AWS CLOUD SOLUTION FOR 2 COMPANY WEBSITES USING A REVERSE PROXY TECHNOLOGY

Starting Off Your AWS Cloud Project
There are few requirements that must be met before you begin:

Properly configure your AWS account and Organization Unit Watch How To Do This Here

Create an AWS Master account. (Also known as Root Account)
Within the Root account, create a sub-account and name it DevOps. (You will need another email address to complete this)
Within the Root account, create an AWS Organization Unit (OU). Name it Dev. (We will launch Dev resources in there)
Move the DevOps account into the Dev OU.
Login to the newly created AWS account using the new email address.
Create a free domain name for your fictitious company at Freenom domain registrar here.

Create a hosted zone in AWS, and map it to your free domain from Freenom

# I created a stepguide to implement [AWS ORGANIZATION](https://github.com/Damdev-95/aws_projects/blob/main/aws-organization.md)

## VPC CREATION

![image](https://user-images.githubusercontent.com/71001536/170456599-0a515010-92e3-4317-ae0c-023cf3885f10.png)

![image](https://user-images.githubusercontent.com/71001536/170463308-6f6efb20-2b7a-4b10-90e5-af42a0a64345.png)

# AMI CREATION fOR BASTION, NGINX, WEBSERVER

* Spin an EC2 INSTANCE for each of them 

* Install the necessary configurations specifed below;

![image](https://user-images.githubusercontent.com/71001536/170511645-6772de1b-6335-4a09-a535-24a2fb2562d5.png)

![image](https://user-images.githubusercontent.com/71001536/170510118-89101bca-541c-4853-b9c6-8ec04d10deba.png)

* Create the image 

![image](https://user-images.githubusercontent.com/71001536/170514780-6ea385f0-26aa-4311-8ca5-0c53c1309c1d.png)


* Create Target Group

![image](https://user-images.githubusercontent.com/71001536/170514345-c072b5df-f6f7-421b-ba37-5ed6f2b1ec65.png)


* Create Load Balancer

![image](https://user-images.githubusercontent.com/71001536/170515893-c2dea4df-9ab2-4305-a752-61f94c791660.png)


* Create Launch Template

* Create Autoscaling Group



