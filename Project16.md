## AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM PART 1

The goal at the end of the project is to deploy AWS resources using Infrasture as a code, which is declarative, easier and scalable.

The diagram is the AWS end-to-end solution for 3 tier Application

![image](https://user-images.githubusercontent.com/71001536/171166981-dea1e815-58a2-46cc-9f6d-39e04a7fbbaf.png)

Due to the capcity of my laptop, am using AWS Cloud9 environment for the development 

# Terraform Installation

* Goto `https://learn.hashicorp.com/tutorials/terraform/install-cli` 

* Copy the following commands to install terraform on Ubuntu Linux distro

```
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common curl
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install terraform
```
* I created an instance profile which assume a role in order for the Cloud9 environment to have access to AWS resources 
* It has both the Trust policy and Permissiom policy 
#  TRUST POLICY 
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "sts:AssumeRole"
            ],
            "Principal": {
                "Service": [
                    "ec2.amazonaws.com"
                ]
            }
        }
    ]
}
```
# PERMISSION POLICY

* I attached the  AWS Managed policy *AdministratorAccess* to the IAM role

* Attach the IAM role to the instance for the Cloud9 environment 

![image](https://user-images.githubusercontent.com/71001536/171359731-59b46e24-fc55-4be7-904a-bfd92f3cf08c.png)

![image](https://user-images.githubusercontent.com/71001536/171359523-6d98c611-6b01-457e-84cc-f9f815503e63.png)

![image](https://user-images.githubusercontent.com/71001536/171360693-82b12949-4508-4c78-af24-e54d15d9aa4b.png)

# Starting Terraform environment 
* First confirm the installed version of terraform 
`terraform --version` 

![image](https://user-images.githubusercontent.com/71001536/171362653-0d3b1532-88f7-45ca-99b8-137074912a03.png)

* Create the provider file  *provider.tf*, *main.tf*, *variabale.tf* in the PBL directory
![image](https://user-images.githubusercontent.com/71001536/171363273-27eb438c-653d-4b0f-a525-bee82c23645a.png)

# provider.tf

```
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
    }
  }
}

provider "aws" {
  region                  = "us-west-2"
}
```

# main.tf

```
resource "aws_vpc" "project_16" {
  cidr_block           = "192.168.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true


  tags = {
    Name = "dev"
  }

}


resource "aws_subnet" "project_16_public_subnet1" {
  vpc_id                  = aws_vpc.project_16.id
  cidr_block              = "192.168.10.0/24"
  map_public_ip_on_launch = true
  availability_zone       = "us-west-2a"

  tags = {
    Name = "dev-public1"

  }
}

resource "aws_subnet" "project_16_public_subnet2" {
  vpc_id                  = aws_vpc.project_16.id
  cidr_block              = "192.168.20.0/24"
  map_public_ip_on_launch = true
  availability_zone       = "us-west-2b"

  tags = {
    Name = "dev-public2"

  }
}

```

* Initailizing terraform files in the *PBL* directory 
```
cd PBL
terraform init 
```
![image](https://user-images.githubusercontent.com/71001536/171362515-18ee84f9-c133-4515-847c-4840b9a38cf1.png)



