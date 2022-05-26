
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


* Create Target Group FOR NGINX AND TOOLING

![image](https://user-images.githubusercontent.com/71001536/170514345-c072b5df-f6f7-421b-ba37-5ed6f2b1ec65.png)


* Create Load Balancer

![image](https://user-images.githubusercontent.com/71001536/170515893-c2dea4df-9ab2-4305-a752-61f94c791660.png)

![image](https://user-images.githubusercontent.com/71001536/170517184-6440b1c3-523f-47f6-ac31-6a776fe681f5.png)

* Add rule to forward to the tooling webserver from INTERNAL-LB

![image](https://user-images.githubusercontent.com/71001536/170517662-a2053879-3ed4-4f9c-b715-6b8cd43b012d.png)

![image](https://user-images.githubusercontent.com/71001536/170517931-735f484a-1f46-4445-900f-6c4e5bd612ad.png)

![image](https://user-images.githubusercontent.com/71001536/170518234-b4a82ddf-5a79-4531-a546-28bfba724ef5.png)

* Create Launch Template


```
#!/bin/bash
mkdir /var/www/
sudo mount -t efs -o tls,accesspoint=fsap-068b735d78fd15e17 fs-03975ba3087bd73fb:/ /var/www/
yum install -y httpd 
systemctl start httpd
systemctl enable httpd
yum module reset php -y
yum module enable php:remi-7.4 -y
yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
systemctl start php-fpm
systemctl enable php-fpm
https://github.com/Damdev-95/project15-config.git
mkdir /var/www/html
cp -R /tooling-1/html/*  /var/www/html/
cd /tooling-1
mysql -h project16-dataabase.cmxpszwyeske.us-east-2.rds.amazonaws.com -u admin -p toolingdb < tooling-db.sql
cd /var/www/html/
touch healthstatus
sed -i "s/$db = mysqli_connect('mysql.tooling.svc.cluster.local', 'admin', 'admin', 'tooling');/$db = mysqli_connect('project16-dataabase.cmxpszwyeske.us-east-2.rds.amazonaws.com', 'admin', 'admin12345', 'toolingdb');/g" functions.php
chcon -t httpd_sys_rw_content_t /var/www/html/ -R
systemctl restart httpd


```
* Create Autoscaling Group



