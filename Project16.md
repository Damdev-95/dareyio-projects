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

* Apply `terraform plan` on the command line

![image](https://user-images.githubusercontent.com/71001536/171371231-698fdd54-9cc6-446e-8edf-ba9c8d74e625.png)

* Use `terraform apply ` to deploy the resources in the *main.tf* file

![image](https://user-images.githubusercontent.com/71001536/171375428-3f8fbd3e-c2f9-49dc-88ba-f40f59f37d2f.png)

* Use `terraform show` or `terraform state list` to display the aws resources that was deployed using terraform 

![image](https://user-images.githubusercontent.com/71001536/171375968-ab8bf4ab-53b2-4c24-83bb-24b546c6ae7b.png)

![image](https://user-images.githubusercontent.com/71001536/171377291-b511d579-fdd1-4585-a06c-ab2983609676.png)

* Usse `terraform destroy` or `terraform destroy -auto-approve` to delete and clean ip AWS resources.

![image](https://user-images.githubusercontent.com/71001536/171378730-a593c035-8b15-404a-9b31-dc0b53d155fe.png)

# REFACTORING CODE FOR SCALABILITY AND EASY CONFIGURATION

* Using *variable.tf* to indicate some reources in the *main.tf* file

```
variable "region" {
  default = "us-west-2"
}

variable "vpc_cidr" {
  default = "192.168.0.0/16"
}

variable "enable_dns_support" {
  default = "true"
}

variable "enable_dns_hostnames" {
  default = "true"
}

variable "enable_classiclink" {
  default = "false"
}

variable "enable_classiclink_dns_support" {
  default = "false"
}
```
* To improve the deployment of the subnets in the availability in the resource blocks using  *Loops & Data sources*
* Get list of availability zones
```
 data "aws_availability_zones" "available" {
        state = "available"
        }
```
* To make use of this new data resource, we will need to introduce a count argument in the subnet block
```
resource "aws_subnet" "public" { 
        count                   = 2
        vpc_id                  = aws_vpc.main.id
        cidr_block              = "172.16.1.0/24"
        map_public_ip_on_launch = true
        availability_zone       = data.aws_availability_zones.available.names[count.index]

    }
```


A closer look at cidrsubnet – this function works like an algorithm to dynamically create a subnet CIDR per AZ. Regardless of the number of subnets created, it takes care of the cidr value per subnet.

# Its parameters are cidrsubnet(prefix, newbits, netnum)

* The *prefix* parameter must be given in CIDR notation, same as for VPC.
* The *newbits* parameter is the number of additional bits with which to extend the prefix. For example, if given a prefix ending with /16 and a newbits value of 4, the resulting subnet address will have length /20
* The *netnum* parameter is a whole number that can be represented as a binary integer with no more than newbits binary digits, which will be used to populate the additional bits added to the prefix

![image](https://user-images.githubusercontent.com/71001536/171381974-70482c42-13c2-4708-8c26-2c360cc50770.png)

# The final problem to solve is removing hard coded count value.
If we cannot hard code a value we want, then we will need a way to dynamically provide the value based on some input. Since the data resource returns all the AZs within a region, it makes sense to count the number of AZs returned and pass that number to the count argument.

To do this, we can introuduce length() function, which basically determines the length of a given list, map, or string.

Since *data.aws_availability_zones.available.names* returns a list like ["eu-central-1a", "eu-central-1b", "eu-central-1c"] we can pass it into a lenght function and get number of the AZs.
length(["eu-central-1a", "eu-central-1b", "eu-central-1c"])

```
 resource "aws_subnet" "public" { 
        count                   = length(data.aws_availability_zones.available.names)
        vpc_id                  = aws_vpc.main.id
        cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
        map_public_ip_on_launch = true
        availability_zone       = data.aws_availability_zones.available.names[count.index]

    }
    
 ```
 # Observations:

What we have now, is sufficient to create the subnet resource required. But if you observe, it is not satisfying our business requirement of just 2 subnets. The length function will return number 3 to the count argument, but what we actually need is 2.
* Declare a variable to store the desired number of public subnets, and set the default value

```variable "preferred_number_of_public_subnets" {
  default = 2
}
```
* Next, update the count argument with a condition. Terraform needs to check first if there is a desired number of subnets. Otherwise, use the data returned by the lenght function.

# Create public subnets

```
resource "aws_subnet" "public" {
  count  = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets   
  vpc_id = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]

}
```
Now lets break it down:

* The first part *var.preferred_number_of_public_subnets == null* checks if the value of the variable is set to null or has some value defined.
* The second part *? and length(data.aws_availability_zones.available.names)* means, if the first part is true, then use this. In other words, if preferred number of public subnets is null (Or not known) then set the value to the data returned by lenght function.
* The third part *: and var.preferred_number_of_public_subnets* means, if the first condition is false, i.e preferred number of public subnets is not null then set the value to whatever is definied in var.preferred_number_of_public_subnets

# Introducing variables.tf & terraform.tfvars

* This enables the use of variables files in the terraform configuration, terraform.tfvars has priority than variables.tf 

# varaibles.tf

```
variable "region" {
  default = "us-west-2"
}

variable "vpc_cidr" {
  default = "192.168.0.0/16"
}

variable "enable_dns_support" {
  default = "true"
}

variable "enable_dns_hostnames" {
  default = "true"
}

variable "enable_classiclink" {
  default = "false"
}

variable "enable_classiclink_dns_support" {
  default = "false"
}

variable "preferred_number_of_public_subnets" {
  default = 2
}
```

# terraform.tfvars

```
region = "us-west-2"

vpc_cidr = "192.168.0.0/16"

enable_dns_support = "true"

enable_dns_hostnames = "true"

enable_classiclink = "false"

enable_classiclink_dns_support = "false"

preferred_number_of_public_subnets = 2
```

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
  region = var.region

}
```

# main.tf

```
resource "aws_vpc" "project_16" {
  cidr_block                     = var.vpc_cidr
  enable_dns_hostnames           = var.enable_dns_hostnames
  enable_dns_support             = var.enable_dns_support
  enable_classiclink             = var.enable_classiclink
  enable_classiclink_dns_support = var.enable_classiclink_dns_support


  tags = {
    Name = "dev"
  }

}


data "aws_availability_zones" "available" {
  state = "available"
}



resource "aws_subnet" "project_16_public_subnet" {
  count                   = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets
  vpc_id                  = aws_vpc.project_16.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 8, count.index)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]
  tags = {
    Name = "dev-public"

  }
}



```
