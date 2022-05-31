## AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM PART 1

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
