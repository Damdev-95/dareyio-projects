## AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM. PART 2

# Networking
# Private subnets & best practices
* Create 4 private subnets keeping in mind following principles:

* Make sure you use variables or length() function to determine the number of AZs
* Use variables and cidrsubnet() function to allocate vpc_cidr for subnets
* Keep variables and resources in separate files for better code structure and readability
* Tags all the resources you have created so far. Explore how to use format() and count functions to automatically tag subnets with its respective number.
* Add tag to the *terraform.tfvars* 

```
tags = {
  Enviroment      = "development" 
  Owner-Email     = "douxtech.ng@gmail.com"
  Managed-By      = "Terraform"
  Billing-Account = "1234567890"
}
```
* Update the *variables.tf* to declare the variable tags used in the format;

```
tags = merge(
    var.tags,
    {
      Name = "Name of the resource"
    },
  )
```
* Includding the following tags on each resources 
```
tags = merge(
    var.tags,
    {
      Name = "Name of the resource"
    },
  )
```

# Internet Gateways & format() function
* Create an Internet Gateway in a separate Terraform file internet_gateway.tf

```
resource "aws_internet_gateway" "project_16_igw" {
  vpc_id = aws_vpc.project_16.id
  tags = merge(
    var.tags,
    {
      Name = format("%s-%s!", aws_vpc.project_16.id,"IG")
    },
  )
}

```

* Using the *format()* function
The first part of the *%s* takes the interpolated value of aws_vpc.project_16.id while the second *%s* appends a literal string IG and finally an exclamation mark is added in the end.
