# AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM. PART 3 – REFACTORING

## Kindly find the github repository for the terraform files in [Project 18](https://github.com/Damdev-95/Project18-PBL)

* Introducing Backend on S3
Each Terraform configuration can specify a backend, which defines where and how operations are performed, where state snapshots are stored, etc.
Take a peek into what the states file looks like. It is basically where terraform stores all the state of the infrastructure in json format.

So far, we have been using the default backend, which is the local backend – it requires no configuration, and the states file is stored locally. This mode can be suitable for learning purposes, but it is not a robust solution, so it is better to store it in some more reliable and durable storage.

The second problem with storing this file locally is that, in a team of multiple DevOps engineers, other engineers will not have access to a state file stored locally on your computer.

To solve this, we will need to configure a backend where the state file can be accessed remotely other DevOps team members. There are plenty of different standard backends supported by Terraform that you can choose from. Since we are already using AWS – we can choose an S3 bucket as a backend.

Another useful option that is supported by S3 backend is State Locking – it is used to lock your state for all operations that could write state. This prevents others from acquiring the lock and potentially corrupting your state. State Locking feature for S3 backend is optional and requires another AWS service – DynamoDB.

Here is our plan to Re-initialize Terraform to use S3 backend:

* Add S3 and DynamoDB resource blocks before deleting the local state file
* Update terraform block to introduce backend and locking
* Re-initialize terraform
* Delete the local tfstate file and check the one in S3 bucket
* Add outputs
* terraform apply
To get to know how lock in DynamoDB works, read the following article

* Create a file and name it backend.tf. Add the below code and replace the name of the S3 bucket you created in Project-16.
# Note: The bucket name may not work for you since buckets are unique globally in AWS, so you must give it a unique name.

```
resource "aws_s3_bucket" "terraform-state" {
  bucket        = "douxtech-project18-2022"
  force_destroy = true
}
resource "aws_s3_bucket_versioning" "version" {
  bucket = aws_s3_bucket.terraform-state.id
  versioning_configuration {
    status = "Enabled"
  }
}
resource "aws_s3_bucket_server_side_encryption_configuration" "first" {
  bucket = aws_s3_bucket.terraform-state.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

```
You must be aware that Terraform stores secret data inside the state files. Passwords, and secret keys processed by resources are always stored in there. Hence, you must consider to always enable encryption. You can see how we achieved that with server_side_encryption_configuration.

Next, we will create a DynamoDB table to handle locks and perform consistency checks. In previous projects, locks were handled with a local file as shown in terraform.tfstate.lock.info. Since we now have a team mindset, causing us to configure S3 as our backend to store state file, we will do the same to handle locking. Therefore, with a cloud storage database like DynamoDB, anyone running Terraform against the same infrastructure can use a central location to control a situation where Terraform is running at the same time from multiple different people.
* Dynamo DB resource for locking and consistency checking:

```
resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"
  attribute {
    name = "LockID"
    type = "S"
  }
}
```
![image](https://user-images.githubusercontent.com/71001536/173820496-08c6f4f0-3ba8-45ea-b7ca-76d45e213877.png)

![image](https://user-images.githubusercontent.com/71001536/173820664-1d734814-b7e7-471a-bdf0-47ab8a4e89b9.png)

![image](https://user-images.githubusercontent.com/71001536/173820789-a4188f10-1ea4-4f27-991b-1aa2da8d2b90.png)


Terraform expects that both S3 bucket and DynamoDB resources are already created before we configure the backend. So, let us run terraform apply to provision resources.

* Configure S3 Backend

```
terraform {
  backend "s3" {
    bucket         = "douxtech-project18-2022"
    key            = "global/s3/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```
Now its time to re-initialize the backend. Run terraform init and confirm you are happy to change the backend by typing yes

![image](https://user-images.githubusercontent.com/71001536/173821522-a0f10b66-61c9-4358-a67a-68a969478e3e.png)

* Add Terraform Output
Before you run terraform apply let us add an output so that the S3 bucket Amazon Resource Names ARN and DynamoDB table name can be displayed.

Create a new file and name it output.tf and add below code.

```
output "s3_bucket_arn" {
  value       = aws_s3_bucket.terraform-state.arn
  description = "The ARN of the S3 bucket"
}
output "dynamodb_table_name" {
  value       = aws_dynamodb_table.terraform_locks.name
  description = "The name of the DynamoDB table"
}
```

![image](https://user-images.githubusercontent.com/71001536/173823323-d8d92cf1-6733-4e7a-ae98-82c106a2bd0a.png)

![image](https://user-images.githubusercontent.com/71001536/173823551-82a10fd3-3e93-4ae0-b0e0-fe517dba6509.png)


* Verify the changes
Before doing anything if you opened AWS now to see what happened you should be able to see the following:

tfstatefile is now inside the S3 bucket
DynamoDB table which we create has an entry which includes state file status

![image](https://user-images.githubusercontent.com/71001536/173824426-f44ef7fa-a937-412d-ae1a-6a0bac0b3007.png)


# WHEN TO USE WORKSPACES OR DIRECTORY?
To separate environments with significant configuration differences, use a directory structure. Use workspaces for environments that do not greatly deviate from each other, to avoid duplication of your configurations. Try both methods in the sections below to help you understand which will serve your infrastructure best.

For now, you can read more about both alternatives here and try both methods yourself, but we will explore them better in next projects.

Security Groups refactoring with dynamic block
For repetitive blocks of code you can use dynamic blocks in Terraform, to get to know more how to use them – watch this video.

# Refactor Security Groups creation with dynamic blocks.

EC2 refactoring with Map and Lookup
Remember, every piece of work you do, always try to make it dynamic to accommodate future changes. Amazon Machine Image (AMI) is a regional service which means it is only available in the region it was created. But what if we change the region later, and want to dynamically pick up AMI IDs based on the available AMIs in that region? This is where we will introduce Map and Lookup functions.

Map uses a key and value pairs as a data structure that can be set as a default type for variables.

```
variable "images" {
    type = "map"
    default = {
        us-east-1 = "image-1234"
        us-west-2 = "image-23834"
    }
}
```
To select an appropriate AMI per region, we will use a lookup function which has following syntax: lookup(map, key, [default]).

Note: A default value is better to be used to avoid failure whenever the map data has no key.

```
resource "aws_instace" "web" {
    ami  = "${lookup(var.images, var.region), "ami-12323"}
}
```

Now, the lookup function will load the variable images using the first parameter. But it also needs to know which of the key-value pairs to use. That is where the second parameter comes in. The key us-east-1 could be specified, but then we will not be doing anything dynamic there, but if we specify the variable for region, it simply resolves to one of the keys. That is why we have used var.region in the second parameter.

Conditional Expressions
If you want to make some decision and choose some resource based on a condition – you shall use Terraform Conditional Expressions.

In general, the syntax is as following: condition ? true_val : false_val

Read following snippet of code and try to understand what it means:

```
resource "aws_db_instance" "read_replica" {
  count               = var.create_read_replica == true ? 1 : 0
  replicate_source_db = aws_db_instance.this.id
}
```
* true #condition equals to ‘if true’
* ? #means, set to ‘1`
* : #means, otherwise, set to ‘0’

# Terraform Modules and best practices to structure your .tf codes

By this time, you might have realized how difficult is to navigate through all the Terraform blocks if they are all written in a single long .tf file. As a DevOps engineer, you must produce reusable and comprehensive IaC code structure, and one of the tool that Terraform provides out of the box is Modules.

Modules serve as containers that allow to logically group Terraform codes for similar resources in the same domain (e.g., Compute, Networking, AMI, etc.). One root module can call other child modules and insert their configurations when applying Terraform config. This concept makes your code structure neater, and it allows different team members to work on different parts of configuration at the same time.

You can also create and publish your modules to Terraform Registry for others to use and use someone’s modules in your projects.

Module is just a collection of .tf and/or .tf.json files in a directory.

This is include in my terraform github repository.

![image](https://user-images.githubusercontent.com/71001536/173785054-0a309422-27c9-4158-9d99-071f6c59a201.png)

![image](https://user-images.githubusercontent.com/71001536/173804586-1dde4808-5b9f-4b59-983a-8e1cac45f4da.png)

![image](https://user-images.githubusercontent.com/71001536/173806281-98ac4b43-2eac-409c-8f39-134421b9a735.png)

![image](https://user-images.githubusercontent.com/71001536/173808060-4bbf1634-125f-4178-aa2f-c38c692ef315.png)

# Final Terraform plan 
![image](https://user-images.githubusercontent.com/71001536/173808389-e11253f6-06d2-468f-8c45-0a86ea9743cd.png)

