# Lab 8 — Terraform Infrastructure as Code

**Date:** July 2026  
**Region:** ap-south-1 (Mumbai)  
**Status:** ✅ Complete  
**Score:** 60% on quiz (new topic — expected)

## What I built
- Installed Terraform and AWS CLI on local machine
- Configured AWS credentials via aws configure
- Wrote 4 Terraform files building complete AWS infrastructure
- Built VPC, subnet, IGW, route table, security group, EC2 with one command
- Verified website running in browser from Terraform output
- Explored terraform state file and state list commands
- Made a change and applied delta update
- Destroyed all resources cleanly with terraform destroy

## Files created
```
lab-01-vpc-ec2/
├── provider.tf    → AWS provider configuration
├── vpc.tf         → VPC, subnet, IGW, route table
├── ec2.tf         → Security group and EC2 instance
└── outputs.tf     → Prints VPC ID, EC2 IP, website URL
```

## The 3 commands that built everything
```bash
terraform init     # download AWS provider plugin
terraform plan     # preview what will be created
terraform apply    # actually build the infrastructure
terraform destroy  # tear everything down cleanly
```

## Full Terraform code

### provider.tf
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "ap-south-1"
}
```

### vpc.tf
```hcl
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true
  tags = { Name = "terraform-vpc" }
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  tags   = { Name = "terraform-igw" }
}

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "ap-south-1a"
  map_public_ip_on_launch = true
  tags = { Name = "terraform-public-subnet" }
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
  tags = { Name = "terraform-public-rt" }
}

resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}
```

### ec2.tf
```hcl
resource "aws_security_group" "web" {
  name        = "terraform-web-sg"
  description = "Allow HTTP and SSH"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
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

  tags = { Name = "terraform-web-sg" }
}

resource "aws_instance" "web" {
  ami                    = "ami-0f5ee92e2d63afc18"
  instance_type          = "t2.micro"
  subnet_id              = aws_subnet.public.id
  vpc_security_group_ids = [aws_security_group.web.id]
  key_name               = "my-aws-key"

  user_data = <<-EOF
    #!/bin/bash
    yum update -y
    yum install -y httpd
    systemctl start httpd
    systemctl enable httpd
    echo "<h1>Built with Terraform by Hemant</h1>" > /var/www/html/index.html
  EOF

  tags = { Name = "terraform-web-server" }
}
```

### outputs.tf
```hcl
output "vpc_id" {
  value = aws_vpc.main.id
}

output "ec2_public_ip" {
  value = aws_instance.web.public_ip
}

output "website_url" {
  value = "http://${aws_instance.web.public_ip}"
}
```

## Key commands used
```bash
# Install and verify
terraform --version
aws --version
aws configure

# Core workflow
terraform init      # downloads provider plugins
terraform plan      # preview changes
terraform apply     # build infrastructure
terraform destroy   # tear everything down

# State management
terraform state list              # list all managed resources
terraform state show aws_instance.web  # details of specific resource
```

## What I learned
- Terraform builds infrastructure from code — reproducible every time
- provider.tf tells Terraform which cloud and region to use
- terraform init downloads the provider plugin from HashiCorp registry
- terraform plan = safe dry run, shows + add, ~ change, - destroy
- terraform apply = executes changes, asks for confirmation
- terraform destroy = clean teardown, no forgotten resources
- State file tracks everything Terraform has built
- Never delete state file — Terraform loses track of infrastructure
- In production state file lives in S3 with DynamoDB locking

## Architecture built
```
Terraform code (local)
    ↓ terraform apply
AWS ap-south-1
    ├── VPC (10.0.0.0/16)
    │   ├── Internet Gateway
    │   ├── Public Subnet (10.0.1.0/24)
    │   ├── Route Table (0.0.0.0/0 → IGW)
    │   └── Security Group (port 80, 22)
    └── EC2 t2.micro
        └── Apache web server
            └── http://YOUR_IP → Built with Terraform by Hemant
```

## Cost cleanup
- terraform destroy run after lab ✅
- All 7 resources destroyed cleanly ✅
- No resources left running ✅

## Screenshot
<img width="1920" height="1020" alt="081618AM072026" src="https://github.com/user-attachments/assets/33af9e96-3482-479f-a804-7c5cc5aa601a" />
<img width="1910" height="873" alt="073327AM072026" src="https://github.com/user-attachments/assets/95a94a41-d56b-4b52-9da0-5ce657dd1109" />
<img width="1910" height="873" alt="073333AM072026" src="https://github.com/user-attachments/assets/7625cd29-16dc-406b-9570-5bfc9d608940" />
<img width="1910" height="873" alt="073810AM072026" src="https://github.com/user-attachments/assets/f6e2cb3f-364a-4f04-b03c-a63b84daf532" />
