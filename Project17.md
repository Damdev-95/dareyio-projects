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
![image](https://user-images.githubusercontent.com/71001536/171882928-6b593e36-c0dc-4fd3-8541-477da89962d6.png)

# SECURITY GROUP
There is order in creating the SG
* External Load balancer : ref public(https,hhtp)
* Bastion : ref admin(ssh)
* REVERSER PROXY NGINX: ref ELB(https) ,bastion(ssh)
* Internal Load balancer: ref nginx(https)
* WEB servers: ref  bastion(ssh), ILB(https)
* Data layer: ref webserver(mysql,nfs), bastion(mysql)

```
# security group for alb, to allow acess from any where for HTTP and HTTPS traffic
resource "aws_security_group" "ext-alb-sg" {
  name        = "ext-alb-sg"
  vpc_id      = aws_vpc.project_16.id
  description = "Allow TLS inbound traffic"

  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "HTTPS"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = format("%s-ext-alb-sg", var.name)
    },
  )

}


# security group for bastion, to allow access into the bastion host from you IP
resource "aws_security_group" "bastion_sg" {
  name        = "vpc_bastion_sg"
  vpc_id      = aws_vpc.project_16.id
  description = "Allow remote SSH connections."

  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = format("%s-Bastion-SG", var.name)
    },
  )
}



#security group for nginx reverse proxy, to allow access only from the extaernal load balancer and bastion instance
resource "aws_security_group" "nginx-sg" {
  name   = "nginx-sg"
  vpc_id = aws_vpc.project_16.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = format("%s-nginx-SG", var.name)
    },
  )
}

resource "aws_security_group_rule" "inbound-nginx-http" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.ext-alb-sg.id
  security_group_id        = aws_security_group.nginx-sg.id
}

resource "aws_security_group_rule" "inbound-bastion-ssh" {
  type                     = "ingress"
  from_port                = 22
  to_port                  = 22
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.bastion_sg.id
  security_group_id        = aws_security_group.nginx-sg.id
}


# security group for ialb, to have acces only from nginx reverser proxy server
resource "aws_security_group" "int-alb-sg" {
  name   = "my-alb-sg"
  vpc_id = aws_vpc.project_16.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = format("%s-int-alb-sg", var.name)
    },
  )

}

resource "aws_security_group_rule" "inbound-ialb-https" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.nginx-sg.id
  security_group_id        = aws_security_group.int-alb-sg.id
}


# security group for webservers, to have access only from the internal load balancer and bastion instance
resource "aws_security_group" "webserver-sg" {
  name   = "my-asg-sg"
  vpc_id = aws_vpc.project_16.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = format("%s-webserver-sg", var.name)
    },
  )

}

resource "aws_security_group_rule" "inbound-web-https" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.int-alb-sg.id
  security_group_id        = aws_security_group.webserver-sg.id
}

resource "aws_security_group_rule" "inbound-web-ssh" {
  type                     = "ingress"
  from_port                = 22
  to_port                  = 22
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.bastion_sg.id
  security_group_id        = aws_security_group.webserver-sg.id
}


# security group for datalayer to alow traffic from websever on nfs and mysql port and bastiopn host on mysql port
resource "aws_security_group" "datalayer-sg" {
  name   = "datalayer-sg"
  vpc_id = aws_vpc.project_16.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = format("%s-datalayer-sg", var.name)
    },
  )
}

resource "aws_security_group_rule" "inbound-nfs-port" {
  type                     = "ingress"
  from_port                = 2049
  to_port                  = 2049
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.webserver-sg.id
  security_group_id        = aws_security_group.datalayer-sg.id
}

resource "aws_security_group_rule" "inbound-mysql-bastion" {
  type                     = "ingress"
  from_port                = 3306
  to_port                  = 3306
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.bastion_sg.id
  security_group_id        = aws_security_group.datalayer-sg.id
}

resource "aws_security_group_rule" "inbound-mysql-webserver" {
  type                     = "ingress"
  from_port                = 3306
  to_port                  = 3306
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.webserver-sg.id
  security_group_id        = aws_security_group.datalayer-sg.id
}
```
![image](https://user-images.githubusercontent.com/71001536/172016842-0a0b82d9-3ef9-4dcb-9f47-bb20807f10f9.png)


# Create the EXTERNAL LOAD BALANCER, INTERNAL LOAD BALANCER, TARGET GROUPS AND LISTENERS
This is done first, in order to attach the public certifcate  fron the ACM

```
# The entire section create a certiface, public zone, and validate the certificate using DNS method

# Create the certificate using a wildcard for all the domains created in douxtech.xyz
resource "aws_acm_certificate" "douxtech" {
  domain_name       = "*.douxtech.xyz"
  validation_method = "DNS"
}

# calling the hosted zone
data "aws_route53_zone" "douxtech" {
  name         = "douxtech.xyz"
  private_zone = false
}

# selecting validation method
resource "aws_route53_record" "douxtech" {
  for_each = {
    for dvo in aws_acm_certificate.douxtech.domain_validation_options : dvo.domain_name => {
      name   = dvo.resource_record_name
      record = dvo.resource_record_value
      type   = dvo.resource_record_type
    }
  }

  allow_overwrite = true
  name            = each.value.name
  records         = [each.value.record]
  ttl             = 60
  type            = each.value.type
  zone_id         = data.aws_route53_zone.douxtech.zone_id
}

# validate the certificate through DNS method
resource "aws_acm_certificate_validation" "douxtech" {
  certificate_arn         = aws_acm_certificate.douxtech.arn
  validation_record_fqdns = [for record in aws_route53_record.douxtech : record.fqdn]
}

# create records for tooling
resource "aws_route53_record" "tooling" {
  zone_id = data.aws_route53_zone.douxtech.zone_id
  name    = "tooling.douxtech.xyz"
  type    = "A"

  alias {
    name                   = aws_lb.ext-alb.dns_name
    zone_id                = aws_lb.ext-alb.zone_id
    evaluate_target_health = true
  }
}


# create records for wordpress
resource "aws_route53_record" "wordpress" {
  zone_id = data.aws_route53_zone.douxtech.zone_id
  name    = "wordpress.douxtech.xyz"
  type    = "A"

  alias {
    name                   = aws_lb.ext-alb.dns_name
    zone_id                = aws_lb.ext-alb.zone_id
    evaluate_target_health = true
  }
}
```

# ACM, Public certificate attached to the External Load balancer 

* This is done to attached a public certificate to all sub-domain for the hosted zone, which enables a secure connection for the domain 
```
# The entire section create a certiface, public zone, and validate the certificate using DNS method

# Create the certificate using a wildcard for all the domains created in douxtech.xyz
resource "aws_acm_certificate" "douxtech" {
  domain_name       = "*.douxtech.xyz"
  validation_method = "DNS"
}

# calling the hosted zone
data "aws_route53_zone" "douxtech" {
  name         = "douxtech.xyz"
  private_zone = false
}

# selecting validation method
resource "aws_route53_record" "douxtech" {
  for_each = {
    for dvo in aws_acm_certificate.douxtech.domain_validation_options : dvo.domain_name => {
      name   = dvo.resource_record_name
      record = dvo.resource_record_value
      type   = dvo.resource_record_type
    }
  }

  allow_overwrite = true
  name            = each.value.name
  records         = [each.value.record]
  ttl             = 60
  type            = each.value.type
  zone_id         = data.aws_route53_zone.douxtech.zone_id
}

# validate the certificate through DNS method
resource "aws_acm_certificate_validation" "douxtech" {
  certificate_arn         = aws_acm_certificate.douxtech.arn
  validation_record_fqdns = [for record in aws_route53_record.douxtech : record.fqdn]
}

# create records for tooling
resource "aws_route53_record" "tooling" {
  zone_id = data.aws_route53_zone.douxtech.zone_id
  name    = "tooling.douxtech.xyz"
  type    = "A"

  alias {
    name                   = aws_lb.ext-alb.dns_name
    zone_id                = aws_lb.ext-alb.zone_id
    evaluate_target_health = true
  }
}


# create records for wordpress
resource "aws_route53_record" "wordpress" {
  zone_id = data.aws_route53_zone.douxtech.zone_id
  name    = "wordpress.douxtech.xyz"
  type    = "A"

  alias {
    name                   = aws_lb.ext-alb.dns_name
    zone_id                = aws_lb.ext-alb.zone_id
    evaluate_target_health = true
  }
}
```
![image](https://user-images.githubusercontent.com/71001536/172044330-19291ca1-7db2-46a8-83fb-7341b1a08ad1.png)

# Depreciated Warning after upgrade 

* I upgrade the terraform using `terraform init -upgrade`
* Then shows some depreciated warning regarding some resources 


![image](https://user-images.githubusercontent.com/71001536/172064948-4b85346c-26c9-4fad-bcd0-771fbf9d4801.png)

![image](https://user-images.githubusercontent.com/71001536/172066456-f41585c4-5ad1-4522-aa6e-bed97206d407.png)

