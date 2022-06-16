
# AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM. PART 4 – TERRAFORM CLOUD

# Terraform Cloud

What Terraform Cloud is and why use it
By now, you should be pretty comfortable writing Terraform code to provision Cloud infrastructure using Configuration Language (HCL). Terraform is an open-source system, that you installed and ran a Virtual Machine (VM) that you had to create, maintain and keep up to date. In Cloud world it is quite common to provide a managed version of an open-source software. Managed means that you do not have to install, configure and maintain it yourself – you just create an account and use it "as A Service".

Terraform Cloud is a managed service that provides you with Terraform CLI to provision infrastructure, either on demand or in response to various events.

By default, Terraform CLI performs operation on the server whene it is invoked, it is perfectly fine if you have a dedicated role who can launch it, but if you have a team who works with Terraform – you need a consistent remote environment with remote workflow and shared state to run Terraform commands.
Migrate your .tf codes to Terraform Cloud
Le us explore how we can migrate our codes to Terraform Cloud and manage our AWS infrastructure from there:

# Create a Terraform Cloud account

Follow this link, create a new account, verify your email and you are ready to start
Most of the features are free, but if you want to explore the difference between free and paid plans – you can check it on this page.

Create an organization
Select "Start from scratch", choose a name for your organization and create it.

* Configure a workspace
Before we begin to configure our workspace – watch this part of the video to better understand the difference between version control workflow, CLI-driven workflow and API-driven workflow and other configurations that we are going to implement.

We will use version control workflow as the most common and recommended way to run Terraform commands triggered from our git repository.

Create a new repository in your GitHub and call it terraform-cloud, push your Terraform codes developed in the previous projects to the repository.

Choose version control workflow and you will be promped to connect your GitHub account to your workspace – follow the prompt and add your newly created repository to the workspace.



Move on to "Configure settings", provide a description for your workspace and leave all the rest settings default, click "Create workspace".

* Configure variables
Terraform Cloud supports two types of variables: environment variables and Terraform variables. Either type can be marked as sensitive, which prevents them from being displayed in the Terraform Cloud web UI and makes them write-only.

Set two environment variables: AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY, set the values that you used in Project 16. These credentials will be used to privision your AWS infrastructure by Terraform Cloud.

After you have set these 2 environment variables – yout Terraform Cloud is all set to apply the codes from GitHub and create all necessary AWS resources.

Now it is time to run our Terrafrom scripts, but in our previous project which was project 18, we talked about using Packer to build our images, and Ansible to configure the infrastructure, so for that we are going to make few changes to our our existing respository from Project 18.
The files that would be Addedd is;

* AMI: for building packer images
Ansible: for Ansible scripts to configure the infrastucture
Before you proceed ensure you have the following tools installed on your local machine;

* packer
* Ansible
Refer to this repository for guidiance on how to refactor your enviroment to meet the new changes above and ensure you go through the README.md file.

Run terraform plan and terraform apply from web console
Switch to "Runs" tab and click on "Queue plan manualy" button. If planning has been successfull, you can proceed and confirm Apply – press "Confirm and apply", provide a comment and "Confirm plan"

Check the logs and verify that everything has run correctly. Note that Terraform Cloud has generated a unique state version that you can open and see the codes applied and the changes made since the last run.

* Test automated terraform plan
By now, you have tried to launch plan and apply manually from Terraform Cloud web console. But since we have an integration with GitHub, the process can be triggered automatically. Try to change something in any of .tf files and look at "Runs" tab again – plan must be launched automatically, but to apply you still need to approve manually. Since provisioning of new Cloud resources might incur significant costs. Even though you can configure "Auto apply", it is always a good idea to verify your plan results before pushing it to apply to avoid any misconfigurations that can cause ‘bill shock’.

![image](https://user-images.githubusercontent.com/71001536/173857652-cdf7810e-4672-4eba-aeae-b2c3889875e7.png)

![image](https://user-images.githubusercontent.com/71001536/173865280-b6873d63-782c-4560-8a05-090adf283244.png)

![image](https://user-images.githubusercontent.com/71001536/174036923-20774203-ed77-42d3-8e74-ac7a6a5c22a4.png)


# Installing Packer on Linux(Ubuntu)

```
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install packer
```

![image](https://user-images.githubusercontent.com/71001536/174040184-2c49b2da-c53e-4243-a4cc-6c2bb62796c5.png)

# EXPORT ENVIRONMENT VARIABLES 
```
# for WINDOWS 
setx AWS_ACCESS_KEY_ID "<YOUR_AWS_ACCESS_KEY_ID>"
setx AWS_SECRET_ACCESS_KEY_ID "<YOUR_AWS_SECRET_ACCESS_KEY>"

# for Linux
export AWS_ACCESS_KEY_ID="<YOUR_AWS_ACCESS_KEY_ID>"
export AWS_SECRET_ACCESS_KEY="<YOUR_AWS_SECRET_ACCESS_KEY>"

#check for existence
printenv | grep -i AWS_ACCESS
```
* create a file for the variables and provider in the ami_folder after the error below

![image](https://user-images.githubusercontent.com/71001536/174069291-a2b49906-9ded-4eed-a8a3-16ab207ab4fa.png)


```
packer {
  required_plugins {
    amazon = {
      version = ">= 0.0.2"
      source  = "github.com/hashicorp/amazon"
    }
  }
}
```

```
variable "region" {
  type    = string
  default = "us-east-1"
}

locals { timestamp = regex_replace(timestamp(), "[- TZ:]", "") }
```

![image](https://user-images.githubusercontent.com/71001536/174068951-8470ca71-5588-4b26-9fe5-35d5c93c393f.png)

![image](https://user-images.githubusercontent.com/71001536/174074360-1a4c66f2-cbd9-4850-b234-dcd889ca4d77.png)

![image](https://user-images.githubusercontent.com/71001536/174088829-8c95bdd6-25b5-40ad-b05a-0cfcf14c2ca7.png)

