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

