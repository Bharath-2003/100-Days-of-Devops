# Project 3: Provision AWS Infrastructure with Terraform

## 🎯 Project Overview

**Difficulty:** Intermediate-Advanced  
**Duration:** 8-10 hours  
**Technologies:** Terraform, AWS, IaC, Remote State

### What You'll Build
A complete AWS infrastructure using Terraform, including VPC, EC2 instances, RDS database, S3 buckets, Load Balancer, Auto Scaling, and proper security configurations with remote state management.

### Learning Objectives
- ✅ Write infrastructure as code with Terraform
- ✅ Provision AWS resources programmatically
- ✅ Implement VPC networking
- ✅ Set up remote state with S3 and DynamoDB
- ✅ Create reusable Terraform modules
- ✅ Implement security best practices
- ✅ Use workspaces for multi-environment
- ✅ Implement auto-scaling and load balancing

### Prerequisites
- AWS account with appropriate permissions
- AWS CLI installed and configured
- Terraform installed (v1.0+)
- Basic understanding of AWS services
- SSH key pair for EC2 access

---

## 📋 Project Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         AWS Cloud                                │
│  ┌────────────────────────────────────────────────────────┐    │
│  │                    VPC (10.0.0.0/16)                    │    │
│  │  ┌──────────────────────────────────────────────────┐  │    │
│  │  │         Public Subnet 1 (10.0.1.0/24)            │  │    │
│  │  │  ┌──────────────┐  ┌──────────────┐             │  │    │
│  │  │  │   NAT GW     │  │  Bastion     │             │  │    │
│  │  │  └──────────────┘  └──────────────┘             │  │    │
│  │  └──────────────────────────────────────────────────┘  │    │
│  │  ┌──────────────────────────────────────────────────┐  │    │
│  │  │         Public Subnet 2 (10.0.2.0/24)            │  │    │
│  │  │  ┌──────────────┐                                │  │    │
│  │  │  │   NAT GW     │                                │  │    │
│  │  │  └──────────────┘                                │  │    │
│  │  └──────────────────────────────────────────────────┘  │    │
│  │                           │                              │    │
│  │  ┌────────────────────────▼──────────────────────────┐  │    │
│  │  │      Application Load Balancer (ALB)              │  │    │
│  │  └────────────────────────┬──────────────────────────┘  │    │
│  │                           │                              │    │
│  │  ┌──────────────────────────────────────────────────┐  │    │
│  │  │        Private Subnet 1 (10.0.11.0/24)           │  │    │
│  │  │  ┌──────────┐  ┌──────────┐  ┌──────────┐      │  │    │
│  │  │  │  EC2-1   │  │  EC2-2   │  │  EC2-3   │      │  │    │
│  │  │  │ (App)    │  │ (App)    │  │ (App)    │      │  │    │
│  │  │  └──────────┘  └──────────┘  └──────────┘      │  │    │
│  │  │         Auto Scaling Group                       │  │    │
│  │  └──────────────────────────────────────────────────┘  │    │
│  │  ┌──────────────────────────────────────────────────┐  │    │
│  │  │        Private Subnet 2 (10.0.12.0/24)           │  │    │
│  │  │  ┌──────────┐  ┌──────────┐  ┌──────────┐      │  │    │
│  │  │  │  EC2-4   │  │  EC2-5   │  │  EC2-6   │      │  │    │
│  │  │  │ (App)    │  │ (App)    │  │ (App)    │      │  │    │
│  │  │  └──────────┘  └──────────┘  └──────────┘      │  │    │
│  │  │         Auto Scaling Group                       │  │    │
│  │  └──────────────────────────────────────────────────┘  │    │
│  │                                                          │    │
│  │  ┌──────────────────────────────────────────────────┐  │    │
│  │  │         Database Subnet 1 (10.0.21.0/24)         │  │    │
│  │  │  ┌──────────────────────────────────┐            │  │    │
│  │  │  │     RDS Primary (PostgreSQL)     │            │  │    │
│  │  │  └──────────────────────────────────┘            │  │    │
│  │  └──────────────────────────────────────────────────┘  │    │
│  │  ┌──────────────────────────────────────────────────┐  │    │
│  │  │         Database Subnet 2 (10.0.22.0/24)         │  │    │
│  │  │  ┌──────────────────────────────────┐            │  │    │
│  │  │  │     RDS Standby (PostgreSQL)     │            │  │    │
│  │  │  └──────────────────────────────────┘            │  │    │
│  │  └──────────────────────────────────────────────────┘  │    │
│  └────────────────────────────────────────────────────────┘    │
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐    │
│  │                    S3 Buckets                           │    │
│  │  - Application Assets                                   │    │
│  │  - Terraform State                                      │    │
│  │  - Logs and Backups                                     │    │
│  └────────────────────────────────────────────────────────┘    │
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐    │
│  │              DynamoDB (State Locking)                   │    │
│  └────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🚀 Step-by-Step Implementation

### Phase 1: Initial Setup

#### Step 1.1: Install Prerequisites

```bash
# Install Terraform
# macOS
brew install terraform

# Linux
wget https://releases.hashicorp.com/terraform/1.6.0/terraform_1.6.0_linux_amd64.zip
unzip terraform_1.6.0_linux_amd64.zip
sudo mv terraform /usr/local/bin/

# Verify installation
terraform version

# Install AWS CLI
# macOS
brew install awscli

# Linux
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Configure AWS CLI
aws configure
# Enter: Access Key ID, Secret Access Key, Region, Output format
```

#### Step 1.2: Create Project Structure

```bash
mkdir -p aws-terraform-infrastructure
cd aws-terraform-infrastructure

# Create directory structure
mkdir -p {modules/{vpc,ec2,rds,alb,s3,security},environments/{dev,staging,prod},scripts}

# Create files
touch {main.tf,variables.tf,outputs.tf,terraform.tfvars,backend.tf}
touch .gitignore README.md
```

#### Step 1.3: Create .gitignore

**File: `.gitignore`**
```
# Terraform
*.tfstate
*.tfstate.*
.terraform/
.terraform.lock.hcl
crash.log
override.tf
override.tf.json
*_override.tf
*_override.tf.json

# Sensitive files
*.tfvars
!terraform.tfvars.example
.env
*.pem
*.key

# IDE
.vscode/
.idea/
*.swp
*.swo
```

---

### Phase 2: Set Up Remote State

#### Step 2.1: Create S3 Bucket and DynamoDB Table (Manual First Time)

**File: `scripts/setup-backend.sh`**
```bash
#!/bin/bash

# Variables
BUCKET_NAME="my-terraform-state-bucket-$(date +%s)"
DYNAMODB_TABLE="terraform-state-lock"
REGION="us-east-1"

# Create S3 bucket
aws s3api create-bucket \
    --bucket $BUCKET_NAME \
    --region $REGION

# Enable versioning
aws s3api put-bucket-versioning \
    --bucket $BUCKET_NAME \
    --versioning-configuration Status=Enabled

# Enable encryption
aws s3api put-bucket-encryption \
    --bucket $BUCKET_NAME \
    --server-side-encryption-configuration '{
        "Rules": [{
            "ApplyServerSideEncryptionByDefault": {
                "SSEAlgorithm": "AES256"
            }
        }]
    }'

# Block public access
aws s3api put-public-access-block \
    --bucket $BUCKET_NAME \
    --public-access-block-configuration \
        BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true

# Create DynamoDB table for state locking
aws dynamodb create-table \
    --table-name $DYNAMODB_TABLE \
    --attribute-definitions AttributeName=LockID,AttributeType=S \
    --key-schema AttributeName=LockID,KeyType=HASH \
    --billing-mode PAY_PER_REQUEST \
    --region $REGION

echo "Backend setup complete!"
echo "S3 Bucket: $BUCKET_NAME"
echo "DynamoDB Table: $DYNAMODB_TABLE"
echo ""
echo "Update backend.tf with these values"
```

#### Step 2.2: Configure Backend

**File: `backend.tf`**
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
  }
}
```

---

### Phase 3: Create Terraform Modules

#### Step 3.1: VPC Module

**File: `modules/vpc/main.tf`**
```hcl
# VPC
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = merge(
    var.tags,
    {
      Name = "${var.project_name}-vpc"
    }
  )
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = merge(
    var.tags,
    {
      Name = "${var.project_name}-igw"
    }
  )
}

# Public Subnets
resource "aws_subnet" "public" {
  count                   = length(var.public_subnet_cidrs)
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnet_cidrs[count.index]
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true

  tags = merge(
    var.tags,
    {
      Name = "${var.project_name}-public-subnet-${count.index + 1}"
      Type = "Public"
    }
  )
}

# Private Subnets
resource "aws_subnet" "private" {
  count             = length(var.private_subnet_cidrs)
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.private_subnet_cidrs[count.index]
  availability_zone = var.availability_zones[count.index]

  tags = merge(
    var.tags,
    {
      Name = "${var.project_name}-private-subnet-${count.index + 1}"
      Type = "Private"
    }
  )
}

# Database Subnets
resource "aws_subnet" "database" {
  count             = length(var.database_subnet_cidrs)
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.database_subnet_cidrs[count.index]
  availability_zone = var.availability_zones[count.index]

  tags = merge(
    var.tags,
    {
      Name = "${var.project_name}-database-subnet-${count.index + 1}"
      Type = "Database"
    }
  )
}

# Elastic IPs for NAT Gateways
resource "aws_eip" "nat" {
  count  = length(var.public_subnet_cidrs)
  domain = "vpc"

  tags = merge(
    var.tags,
    {
      Name = "${var.project_name}-nat-eip-${count.index + 1}"
    }
  )

  depends_on = [aws_internet_gateway.main]
}

# NAT Gateways
resource "aws_nat_gateway" "main" {
  count         = length(var.public_subnet_cidrs)
  allocation_id = aws_eip.nat[count.index].id
  subnet_id     = aws_subnet.public[count.index].id

  tags = merge(
    var.tags,
    {
      Name = "${var.project_name}-nat-gw-${count.index + 1}"
    }
  )

  depends_on = [aws_internet_gateway.main]
}

# Public Route Table
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = merge(
    var.tags,
    {
      Name = "${var.project_name}-public-rt"
    }
  )
}

# Private Route Tables
resource "aws_route_table" "private" {
  count  = length(var.private_subnet_cidrs)
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main[count.index].id
  }

  tags = merge(
    var.tags,
    {
      Name = "${var.project_name}-private-rt-${count.index + 1}"
    }
  )
}

# Route Table Associations
resource "aws_route_table_association" "public" {
  count          = length(var.public_subnet_cidrs)
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "private" {
  count          = length(var.private_subnet_cidrs)
  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private[count.index].id
}
```

**File: `modules/vpc/variables.tf`**
```hcl
variable "project_name" {
  description = "Project name"
  type        = string
}

variable "vpc_cidr" {
  description = "VPC CIDR block"
  type        = string
}

variable "public_subnet_cidrs" {
  description = "Public subnet CIDR blocks"
  type        = list(string)
}

variable "private_subnet_cidrs" {
  description = "Private subnet CIDR blocks"
  type        = list(string)
}

variable "database_subnet_cidrs" {
  description = "Database subnet CIDR blocks"
  type        = list(string)
}

variable "availability_zones" {
  description = "Availability zones"
  type        = list(string)
}

variable "tags" {
  description = "Common tags"
  type        = map(string)
  default     = {}
}
```

**File: `modules/vpc/outputs.tf`**
```hcl
output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}

output "public_subnet_ids" {
  description = "Public subnet IDs"
  value       = aws_subnet.public[*].id
}

output "private_subnet_ids" {
  description = "Private subnet IDs"
  value       = aws_subnet.private[*].id
}

output "database_subnet_ids" {
  description = "Database subnet IDs"
  value       = aws_subnet.database[*].id
}

output "nat_gateway_ids" {
  description = "NAT Gateway IDs"
  value       = aws_nat_gateway.main[*].id
}
```

#### Step 3.2: Security Groups Module

**File: `modules/security/main.tf`**
```hcl
# ALB Security Group
resource "aws_security_group" "alb" {
  name_prefix = "${var.project_name}-alb-sg"
  description = "Security group for Application Load Balancer"
  vpc_id      = var.vpc_id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "Allow HTTP from anywhere"
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "Allow HTTPS from anywhere"
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
    description = "Allow all outbound traffic"
  }

  tags = merge(
    var.tags,
    {
      Name = "${var.project_name}-alb-sg"
    }
  )
}

# Application Security Group
resource "aws_security_group" "app" {
  name_prefix = "${var.project_name}-app-sg"
  description = "Security group for application servers"
  vpc_id      = var.vpc_id

  ingress {
    from_port       = 3000
    to_port         = 3000
    protocol        = "tcp"
    security_groups = [aws_security_group.alb.id]
    description     = "Allow traffic from ALB"
  }

  ingress {
    from_port       = 22
    to_port         = 22
    protocol        = "tcp"
    security_groups = [aws_security_group.bastion.id]
    description     = "Allow SSH from bastion"
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
    description = "Allow all outbound traffic"
  }

  tags = merge(
    var.tags,
    {
      Name = "${var.project_name}-app-sg"
    }
  )
}

# Database Security Group
resource "aws_security_group" "database" {
  name_prefix = "${var.project_name}-db-sg"
  description = "Security group for RDS database"
  vpc_id      = var.vpc_id

  ingress {
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.app.id]
    description     = "Allow PostgreSQL from app servers"
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
    description = "Allow all outbound traffic"
  }

  tags = merge(
    var.tags,
    {
      Name = "${var.project_name}-db-sg"
    }
  )
}

# Bastion Security Group
resource "aws_security_group" "bastion" {
  name_prefix = "${var.project_name}-bastion-sg"
  description = "Security group for bastion host"
  vpc_id      = var.vpc_id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = var.allowed_ssh_cidrs
    description = "Allow SSH from specific IPs"
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
    description = "Allow all outbound traffic"
  }

  tags = merge(
    var.tags,
    {
      Name = "${var.project_name}-bastion-sg"
    }
  )
}
```

---

### Phase 4: Main Configuration

#### Step 4.1: Root Main Configuration

**File: `main.tf`**
```hcl
terraform {
  required_version = ">= 1.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region

  default_tags {
    tags = {
      Project     = var.project_name
      Environment = var.environment
      ManagedBy   = "Terraform"
    }
  }
}

# VPC Module
module "vpc" {
  source = "./modules/vpc"

  project_name          = var.project_name
  vpc_cidr              = var.vpc_cidr
  public_subnet_cidrs   = var.public_subnet_cidrs
  private_subnet_cidrs  = var.private_subnet_cidrs
  database_subnet_cidrs = var.database_subnet_cidrs
  availability_zones    = var.availability_zones
  tags                  = var.tags
}

# Security Groups Module
module "security" {
  source = "./modules/security"

  project_name      = var.project_name
  vpc_id            = module.vpc.vpc_id
  allowed_ssh_cidrs = var.allowed_ssh_cidrs
  tags              = var.tags
}

# Data source for latest Amazon Linux 2 AMI
data "aws_ami" "amazon_linux_2" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

# Launch Template
resource "aws_launch_template" "app" {
  name_prefix   = "${var.project_name}-lt-"
  image_id      = data.aws_ami.amazon_linux_2.id
  instance_type = var.instance_type

  vpc_security_group_ids = [module.security.app_sg_id]

  user_data = base64encode(templatefile("${path.module}/scripts/user-data.sh", {
    app_name = var.project_name
  }))

  tag_specifications {
    resource_type = "instance"
    tags = merge(
      var.tags,
      {
        Name = "${var.project_name}-app-instance"
      }
    )
  }
}

# Auto Scaling Group
resource "aws_autoscaling_group" "app" {
  name                = "${var.project_name}-asg"
  vpc_zone_identifier = module.vpc.private_subnet_ids
  target_group_arns   = [aws_lb_target_group.app.arn]
  health_check_type   = "ELB"
  health_check_grace_period = 300

  min_size         = var.asg_min_size
  max_size         = var.asg_max_size
  desired_capacity = var.asg_desired_capacity

  launch_template {
    id      = aws_launch_template.app.id
    version = "$Latest"
  }

  tag {
    key                 = "Name"
    value               = "${var.project_name}-asg-instance"
    propagate_at_launch = true
  }
}

# Application Load Balancer
resource "aws_lb" "app" {
  name               = "${var.project_name}-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [module.security.alb_sg_id]
  subnets            = module.vpc.public_subnet_ids

  enable_deletion_protection = false

  tags = merge(
    var.tags,
    {
      Name = "${var.project_name}-alb"
    }
  )
}

# Target Group
resource "aws_lb_target_group" "app" {
  name     = "${var.project_name}-tg"
  port     = 3000
  protocol = "HTTP"
  vpc_id   = module.vpc.vpc_id

  health_check {
    enabled             = true
    healthy_threshold   = 2
    interval            = 30
    matcher             = "200"
    path                = "/health"
    port                = "traffic-port"
    protocol            = "HTTP"
    timeout             = 5
    unhealthy_threshold = 2
  }

  tags = merge(
    var.tags,
    {
      Name = "${var.project_name}-tg"
    }
  )
}

# ALB Listener
resource "aws_lb_listener" "app" {
  load_balancer_arn = aws_lb.app.arn
  port              = "80"
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.app.arn
  }
}
```

**File: `variables.tf`**
```hcl
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "project_name" {
  description = "Project name"
  type        = string
}

variable "environment" {
  description = "Environment (dev/staging/prod)"
  type        = string
}

variable "vpc_cidr" {
  description = "VPC CIDR block"
  type        = string
  default     = "10.0.0.0/16"
}

variable "public_subnet_cidrs" {
  description = "Public subnet CIDR blocks"
  type        = list(string)
  default     = ["10.0.1.0/24", "10.0.2.0/24"]
}

variable "private_subnet_cidrs" {
  description = "Private subnet CIDR blocks"
  type        = list(string)
  default     = ["10.0.11.0/24", "10.0.12.0/24"]
}

variable "database_subnet_cidrs" {
  description = "Database subnet CIDR blocks"
  type        = list(string)
  default     = ["10.0.21.0/24", "10.0.22.0/24"]
}

variable "availability_zones" {
  description = "Availability zones"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b"]
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}

variable "asg_min_size" {
  description = "Auto Scaling Group minimum size"
  type        = number
  default     = 2
}

variable "asg_max_size" {
  description = "Auto Scaling Group maximum size"
  type        = number
  default     = 6
}

variable "asg_desired_capacity" {
  description = "Auto Scaling Group desired capacity"
  type        = number
  default     = 3
}

variable "allowed_ssh_cidrs" {
  description = "CIDR blocks allowed to SSH"
  type        = list(string)
  default     = ["0.0.0.0/0"]
}

variable "tags" {
  description = "Common tags"
  type        = map(string)
  default     = {}
}
```

**File: `outputs.tf`**
```hcl
output "vpc_id" {
  description = "VPC ID"
  value       = module.vpc.vpc_id
}

output "alb_dns_name" {
  description = "ALB DNS name"
  value       = aws_lb.app.dns_name
}

output "alb_zone_id" {
  description = "ALB Zone ID"
  value       = aws_lb.app.zone_id
}
```

**File: `terraform.tfvars.example`**
```hcl
project_name = "myapp"
environment  = "dev"
aws_region   = "us-east-1"

vpc_cidr              = "10.0.0.0/16"
public_subnet_cidrs   = ["10.0.1.0/24", "10.0.2.0/24"]
private_subnet_cidrs  = ["10.0.11.0/24", "10.0.12.0/24"]
database_subnet_cidrs = ["10.0.21.0/24", "10.0.22.0/24"]
availability_zones    = ["us-east-1a", "us-east-1b"]

instance_type        = "t3.micro"
asg_min_size         = 2
asg_max_size         = 6
asg_desired_capacity = 3

allowed_ssh_cidrs = ["YOUR_IP/32"]

tags = {
  Owner = "DevOps Team"
  Cost  = "Project"
}
```

---

## 🧪 Deployment Steps

### Step 1: Initialize Terraform

```bash
# Initialize Terraform
terraform init

# Validate configuration
terraform validate

# Format code
terraform fmt -recursive
```

### Step 2: Plan Infrastructure

```bash
# Create execution plan
terraform plan -out=tfplan

# Review plan
terraform show tfplan
```

### Step 3: Apply Configuration

```bash
# Apply changes
terraform apply tfplan

# Or apply directly (with approval)
terraform apply

# Apply with auto-approve (use with caution)
terraform apply -auto-approve
```

### Step 4: Verify Deployment

```bash
# Get outputs
terraform output

# Get ALB DNS
terraform output alb_dns_name

# Test application
curl http://$(terraform output -raw alb_dns_name)
```

---

## 📊 Testing and Validation

### Test 1: Infrastructure Validation

```bash
# Validate all resources
terraform state list

# Show specific resource
terraform state show aws_lb.app

# Check resource dependencies
terraform graph | dot -Tpng > graph.png
```

### Test 2: Application Testing

```bash
# Get ALB DNS
ALB_DNS=$(terraform output -raw alb_dns_name)

# Test endpoints
curl http://$ALB_DNS/
curl http://$ALB_DNS/health
curl http://$ALB_DNS/api/users

# Load testing
ab -n 1000 -c 10 http://$ALB_DNS/
```

### Test 3: Auto Scaling

```bash
# Generate load
for i in {1..1000}; do curl http://$ALB_DNS/ & done

# Watch ASG
watch -n 5 'aws autoscaling describe-auto-scaling-groups \
  --auto-scaling-group-names myapp-asg \
  --query "AutoScalingGroups[0].Instances[*].[InstanceId,HealthStatus,LifecycleState]" \
  --output table'
```

---

## 🎯 Success Criteria

- [ ] Terraform initializes successfully
- [ ] All resources created without errors
- [ ] VPC and subnets configured correctly
- [ ] ALB accessible from internet
- [ ] Auto Scaling Group running desired instances
- [ ] Application responds to requests
- [ ] Remote state stored in S3
- [ ] State locking working with DynamoDB

---

## 🧹 Cleanup

```bash
# Destroy all resources
terraform destroy

# Destroy with auto-approve
terraform destroy -auto-approve

# Destroy specific resource
terraform destroy -target=aws_lb.app
```

---

## 📚 Additional Resources

- [Terraform Documentation](https://www.terraform.io/docs/)
- [AWS Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [Terraform Best Practices](https://www.terraform-best-practices.com/)

---

## 🎓 What You Learned

✅ Infrastructure as Code with Terraform  
✅ AWS VPC networking  
✅ Remote state management  
✅ Terraform modules  
✅ Auto Scaling and Load Balancing  
✅ Security best practices  
✅ Multi-environment setup  

---

## 🚀 Next Steps

1. **Project 4**: Set up Prometheus + Grafana monitoring
2. Add RDS database
3. Implement blue-green deployment
4. Add CloudFront CDN
5. Implement disaster recovery

---

**Project Status:** ✅ Complete  
**Last Updated:** 2024-01-16