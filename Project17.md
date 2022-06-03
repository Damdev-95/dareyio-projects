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
  Billing-Account = "014285054687"
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

# PRIVATE SUBNET 
```
resource "aws_subnet" "private" {
  count                   = var.preferred_number_of_private_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_private_subnets
  vpc_id                  = aws_vpc.project_16.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 8, count.index +2)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]
  tags = merge(
    var.tags,
    {
       Name = format("%s-PrivateSubnet-%s", var.name, count.index)
    },
  )
}

```

# NAT Gateways

* Create 1 NAT Gateways and 1 Elastic IP (EIP) addresses
```
resource "aws_eip" "nat_eip" {
  vpc        = true
  depends_on = [aws_internet_gateway.igw]

  tags = merge(
    var.tags,
    {
      Name = format("%s-EIP", var.name)
    },
  )
}

resource "aws_nat_gateway" "natgw" {
  allocation_id = aws_eip.nat_eip.id
  subnet_id     = element(aws_subnet.public.*.id, 0)
  depends_on    = [aws_internet_gateway.igw]

  tags = merge(
    var.tags,
    {
      Name = format("%s-NAT", var.name)
    },
  )
}
```
# AWS ROUTES
* Ensure they are properly tagged.

* aws_route_table
* aws_route
* aws_route_table_association
```
resource "aws_route_table" "private-rtb" {
  vpc_id = aws_vpc.project_16.id

  tags = merge(
    var.tags,
    {
      Name = format("%s-Private-Route-Table", var.name)
    },
  )
}

# create route for the private route table and attatch a nat gateway to it
resource "aws_route" "private-rtb-route" {
  route_table_id         = aws_route_table.private-rtb.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_nat_gateway.natgw.id
}


# associate all private subnets to the private route table
resource "aws_route_table_association" "private-subnets-assoc" {
  count          = length(aws_subnet.private[*].id)
  subnet_id      = element(aws_subnet.private[*].id, count.index)
  route_table_id = aws_route_table.private-rtb.id
}



# create route table for the public subnets
resource "aws_route_table" "public-rtb" {
  vpc_id = aws_vpc.project_16.id

  tags = merge(
    var.tags,
    {
      Name = format("%s-Public-Route-Table", var.name)
    },
  )
}

# create route for the public route table and attach the internet gateway
resource "aws_route" "public-rtb-route" {
  route_table_id         = aws_route_table.public-rtb.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_internet_gateway.ig.id
}

# associate all public subnets to the public route table
resource "aws_route_table_association" "public-subnets-assoc" {
  count          = length(aws_subnet.public[*].id)
  subnet_id      = element(aws_subnet.public[*].id, count.index)
  route_table_id = aws_route_table.public-rtb.id
}
```
# AWS Identity and Access Management
# IaM and Roles
We want to pass an IAM role our EC2 instances to give them access to some specific resources, so we need to do the following:
Trust policy: This include the temporary credentials to assume the role.
Permissiom policy: This include the permission to the aws resources.

# Create AssumeRole
Assume Role uses Security Token Service (STS) API that returns a set of temporary security credentials that you can use to access AWS resources that you might not normally have access to. These temporary credentials consist of an access key ID, a secret access key, and a security token. Typically, you use AssumeRole within your account or for cross-account access.
```
resource "aws_iam_role" "ec2_instance_role" {
  name = "ec2_instance_role"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Sid    = ""
        Principal = {
          Service = "ec2.amazonaws.com"
        }
      },
    ]
  })
  tags = merge(
    var.tags,
    {
      Name = "aws assume role"
    },
  )
}

resource "aws_iam_policy" "policy" {
  name        = "ec2_instance_policy"
  description = "A test policy"
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = [
          "ec2:Describe*",
        ]
        Effect   = "Allow"
        Resource = "*"
      },
    ]

  })

  tags = merge(
    var.tags,
    {
      Name =  "aws assume policy"
    },
  )
}

resource "aws_iam_role_policy_attachment" "test-attach" {
  role       = aws_iam_role.ec2_instance_role.name
  policy_arn = aws_iam_policy.policy.arn
}

resource "aws_iam_instance_profile" "ip" {
  name = "aws_instance_profile_test"
  role = aws_iam_role.ec2_instance_role.name
}

```
