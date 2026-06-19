# Terraform for DevOps

## 🎯 Overview

Terraform is an Infrastructure as Code (IaC) tool that enables you to define and provision infrastructure using declarative configuration files. It supports multiple cloud providers and on-premises infrastructure, making it essential for modern DevOps practices.

**Duration:** 2 weeks
**Time Investment:** 4-5 hours/day
**Difficulty:** Intermediate to Advanced
**Prerequisites:** Basic cloud knowledge (AWS/Azure/GCP), understanding of infrastructure concepts

---

## 📚 What You'll Learn

### Week 1: Terraform Fundamentals
- Terraform basics and HCL syntax
- Providers and resources
- Variables and outputs
- State management
- Modules

### Week 2: Advanced Terraform
- Remote state and backends
- Workspaces
- Dynamic blocks and expressions
- Testing and validation
- CI/CD integration

---

## 🎓 Important Concepts

### 1. Terraform Workflow

```
Terraform Workflow
┌─────────────────────────────────────┐
│         1. Write                    │
│    Define infrastructure in         │
│    .tf configuration files          │
└────────────┬────────────────────────┘
             │
             ▼
┌─────────────────────────────────────┐
│         2. Init                     │
│    terraform init                   │
│    - Download providers             │
│    - Initialize backend             │
└────────────┬────────────────────────┘
             │
             ▼
┌─────────────────────────────────────┐
│         3. Plan                     │
│    terraform plan                   │
│    - Preview changes                │
│    - Validate configuration         │
└────────────┬────────────────────────┘
             │
             ▼
┌─────────────────────────────────────┐
│         4. Apply                    │
│    terraform apply                  │
│    - Create/update resources        │
│    - Update state file              │
└────────────┬────────────────────────┘
             │
             ▼
┌─────────────────────────────────────┐
│         5. Destroy (optional)       │
│    terraform destroy                │
│    - Remove all resources           │
└─────────────────────────────────────┘

Key Commands:
├─ terraform init      # Initialize working directory
├─ terraform validate  # Validate configuration
├─ terraform plan      # Preview changes
├─ terraform apply     # Apply changes
├─ terraform destroy   # Destroy infrastructure
├─ terraform fmt       # Format code
├─ terraform show      # Show current state
├─ terraform output    # Show outputs
└─ terraform state     # State management
```

---

### 2. HCL Syntax Basics

```hcl
# Terraform block - specify version and providers
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# Provider configuration
provider "aws" {
  region = var.aws_region
}

# Resource definition
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = var.instance_type
  
  tags = {
    Name = "${var.project_name}-web-server"
  }
}

# Variable definition
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

# Output definition
output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.web.id
}

# Data source
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
}

# Local values
locals {
  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}
```

---

### 3. Resource Management

```hcl
# Basic resource
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
}

# Resource with count
resource "aws_instance" "web" {
  count = 3
  
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  
  tags = {
    Name = "web-server-${count.index}"
  }
}

# Resource with for_each
resource "aws_instance" "servers" {
  for_each = {
    web = "t3.micro"
    app = "t3.small"
    db  = "t3.medium"
  }
  
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = each.value
  
  tags = {
    Name = "${each.key}-server"
  }
}

# Lifecycle rules
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  
  lifecycle {
    create_before_destroy = true
    prevent_destroy       = false
    ignore_changes        = [tags]
  }
}

# Dependencies
resource "aws_eip" "example" {
  instance = aws_instance.example.id
  vpc      = true
  
  depends_on = [aws_internet_gateway.example]
}
```

---

### 4. Variables and Outputs

```hcl
# String variable
variable "region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

# Number variable
variable "instance_count" {
  description = "Number of instances"
  type        = number
  default     = 3
}

# Boolean variable
variable "enable_monitoring" {
  description = "Enable monitoring"
  type        = bool
  default     = true
}

# List variable
variable "availability_zones" {
  description = "Availability zones"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b"]
}

# Map variable
variable "instance_types" {
  description = "Instance types by environment"
  type        = map(string)
  default = {
    dev  = "t3.micro"
    prod = "t3.large"
  }
}

# Object variable
variable "server_config" {
  description = "Server configuration"
  type = object({
    instance_type = string
    disk_size     = number
    enable_backup = bool
  })
  default = {
    instance_type = "t3.micro"
    disk_size     = 20
    enable_backup = true
  }
}

# Variable validation
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  
  validation {
    condition     = contains(["t3.micro", "t3.small", "t3.medium"], var.instance_type)
    error_message = "Instance type must be t3.micro, t3.small, or t3.medium."
  }
}

# Outputs
output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.web.id
}

output "public_ip" {
  description = "Public IP address"
  value       = aws_instance.web.public_ip
  sensitive   = false
}

output "connection_string" {
  description = "Database connection string"
  value       = "postgresql://${aws_db_instance.main.endpoint}"
  sensitive   = true
}
```

---

### 5. Modules

```
Module Structure:
my-module/
├── main.tf          # Main configuration
├── variables.tf     # Input variables
├── outputs.tf       # Output values
├── versions.tf      # Provider versions
└── README.md        # Documentation
```

**Creating a Module:**
```hcl
# modules/vpc/main.tf
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  
  tags = merge(
    var.tags,
    {
      Name = var.vpc_name
    }
  )
}

resource "aws_subnet" "public" {
  count = length(var.public_subnet_cidrs)
  
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnet_cidrs[count.index]
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true
  
  tags = {
    Name = "${var.vpc_name}-public-${count.index + 1}"
  }
}

# modules/vpc/variables.tf
variable "vpc_name" {
  description = "Name of the VPC"
  type        = string
}

variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "public_subnet_cidrs" {
  description = "CIDR blocks for public subnets"
  type        = list(string)
}

variable "availability_zones" {
  description = "Availability zones"
  type        = list(string)
}

variable "tags" {
  description = "Tags to apply"
  type        = map(string)
  default     = {}
}

# modules/vpc/outputs.tf
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}

output "public_subnet_ids" {
  description = "IDs of public subnets"
  value       = aws_subnet.public[*].id
}
```

**Using a Module:**
```hcl
# main.tf
module "vpc" {
  source = "./modules/vpc"
  
  vpc_name             = "production-vpc"
  vpc_cidr             = "10.0.0.0/16"
  public_subnet_cidrs  = ["10.0.1.0/24", "10.0.2.0/24"]
  availability_zones   = ["us-east-1a", "us-east-1b"]
  
  tags = {
    Environment = "production"
  }
}

# Use module outputs
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  subnet_id     = module.vpc.public_subnet_ids[0]
}

# Using public modules
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"
  
  name = "my-vpc"
  cidr = "10.0.0.0/16"
  
  azs             = ["us-east-1a", "us-east-1b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]
}
```

---

### 6. State Management

```hcl
# S3 Backend (AWS)
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
  }
}

# Azure Backend
terraform {
  backend "azurerm" {
    resource_group_name  = "terraform-state-rg"
    storage_account_name = "tfstatestorage"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}

# GCS Backend (Google Cloud)
terraform {
  backend "gcs" {
    bucket = "my-terraform-state"
    prefix = "prod"
  }
}
```

**State Commands:**
```bash
# List resources
terraform state list

# Show resource details
terraform state show aws_instance.web

# Move resource
terraform state mv aws_instance.old aws_instance.new

# Remove resource from state
terraform state rm aws_instance.web

# Import existing resource
terraform import aws_instance.web i-1234567890abcdef0

# Pull remote state
terraform state pull > terraform.tfstate

# Push local state
terraform state push terraform.tfstate
```

---

### 7. Dynamic Blocks and Expressions

```hcl
# Dynamic blocks
resource "aws_security_group" "example" {
  name   = "example-sg"
  vpc_id = aws_vpc.main.id
  
  dynamic "ingress" {
    for_each = var.ingress_rules
    
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}

# Conditional expressions
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = var.environment == "prod" ? "t3.large" : "t3.micro"
}

# For expressions
locals {
  uppercase_names = [for name in var.names : upper(name)]
  large_instances = [for i in var.instances : i if i.size > 100]
  
  instance_types = {
    for k, v in var.servers : k => v.instance_type
  }
}

# Common functions
locals {
  # String functions
  upper_name = upper(var.name)
  lower_name = lower(var.name)
  trimmed    = trim(var.text)
  
  # Collection functions
  merged_tags = merge(var.default_tags, var.custom_tags)
  unique_list = distinct(var.items)
  list_length = length(var.items)
  
  # Numeric functions
  max_value = max(var.numbers...)
  min_value = min(var.numbers...)
  
  # File functions
  file_content = file("${path.module}/config.txt")
  template     = templatefile("${path.module}/template.tpl", {
    name = var.name
  })
}
```

---

## 🔨 Hands-On Exercises

### Exercise 1: AWS VPC with EC2 (60 minutes)

```hcl
# main.tf
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
}

# VPC
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  
  tags = {
    Name = "${var.project_name}-vpc"
  }
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = {
    Name = "${var.project_name}-igw"
  }
}

# Public Subnet
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnet_cidr
  map_public_ip_on_launch = true
  
  tags = {
    Name = "${var.project_name}-public-subnet"
  }
}

# Route Table
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
  
  tags = {
    Name = "${var.project_name}-public-rt"
  }
}

# Route Table Association
resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}

# Security Group
resource "aws_security_group" "web" {
  name        = "${var.project_name}-web-sg"
  description = "Security group for web server"
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
  
  tags = {
    Name = "${var.project_name}-web-sg"
  }
}

# EC2 Instance
resource "aws_instance" "web" {
  ami                    = var.ami_id
  instance_type          = var.instance_type
  subnet_id              = aws_subnet.public.id
  vpc_security_group_ids = [aws_security_group.web.id]
  
  user_data = <<-EOF
              #!/bin/bash
              yum update -y
              yum install -y httpd
              systemctl start httpd
              systemctl enable httpd
              echo "<h1>Hello from Terraform!</h1>" > /var/www/html/index.html
              EOF
  
  tags = {
    Name = "${var.project_name}-web-server"
  }
}

# variables.tf
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "project_name" {
  description = "Project name"
  type        = string
  default     = "myproject"
}

variable "vpc_cidr" {
  description = "VPC CIDR block"
  type        = string
  default     = "10.0.0.0/16"
}

variable "public_subnet_cidr" {
  description = "Public subnet CIDR"
  type        = string
  default     = "10.0.1.0/24"
}

variable "ami_id" {
  description = "AMI ID"
  type        = string
}

variable "instance_type" {
  description = "Instance type"
  type        = string
  default     = "t3.micro"
}

# outputs.tf
output "vpc_id" {
  value = aws_vpc.main.id
}

output "instance_public_ip" {
  value = aws_instance.web.public_ip
}

output "instance_url" {
  value = "http://${aws_instance.web.public_ip}"
}
```

---

### Exercise 2: Multi-Environment Setup (60 minutes)

```bash
# Directory structure
terraform-project/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   │   ├── main.tf
│   │   └── terraform.tfvars
│   └── prod/
│       ├── main.tf
│       └── terraform.tfvars
└── modules/
    └── app/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

```hcl
# modules/app/main.tf
resource "aws_instance" "app" {
  count = var.instance_count
  
  ami           = var.ami_id
  instance_type = var.instance_type
  
  tags = {
    Name        = "${var.environment}-app-${count.index + 1}"
    Environment = var.environment
  }
}

# modules/app/variables.tf
variable "environment" {
  description = "Environment name"
  type        = string
}

variable "instance_count" {
  description = "Number of instances"
  type        = number
}

variable "instance_type" {
  description = "Instance type"
  type        = string
}

variable "ami_id" {
  description = "AMI ID"
  type        = string
}

# environments/dev/main.tf
module "app" {
  source = "../../modules/app"
  
  environment    = "dev"
  instance_count = 1
  instance_type  = "t3.micro"
  ami_id         = var.ami_id
}

# environments/dev/terraform.tfvars
ami_id = "ami-0c55b159cbfafe1f0"
```

---

### Exercise 3: CI/CD with GitHub Actions (45 minutes)

```yaml
# .github/workflows/terraform.yml
name: Terraform

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  terraform:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: 1.6.0
      
      - name: Terraform Format
        run: terraform fmt -check
      
      - name: Terraform Init
        run: terraform init
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      
      - name: Terraform Validate
        run: terraform validate
      
      - name: Terraform Plan
        run: terraform plan
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      
      - name: Terraform Apply
        if: github.ref == 'refs/heads/main'
        run: terraform apply -auto-approve
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

---

## 🎯 Practice Projects

### Project 1: Three-Tier Web Application
Deploy complete web application:
- VPC with public/private subnets
- Application Load Balancer
- Auto Scaling Group
- RDS database
- S3 for static assets

### Project 2: Kubernetes Cluster
Create managed Kubernetes:
- EKS/AKS/GKE cluster
- Node groups
- Networking setup
- Storage classes
- Monitoring integration

### Project 3: Multi-Cloud Infrastructure
Deploy across clouds:
- AWS, Azure, GCP resources
- Unified networking
- Cross-cloud monitoring
- Disaster recovery

### Project 4: Serverless Application
Build serverless stack:
- Lambda/Functions
- API Gateway
- DynamoDB/CosmosDB
- S3/Blob Storage
- CloudWatch/Monitor

---

## 📝 Best Practices

### 1. Code Organization
- Use modules for reusability
- Separate environments
- Version control everything
- Document configurations
- Use consistent naming
- Implement code reviews

### 2. State Management
- Use remote state
- Enable state locking
- Backup state files
- Never commit state
- Use workspaces wisely
- Implement encryption

### 3. Security
- Use secrets management
- Implement least privilege
- Enable encryption
- Scan for vulnerabilities
- Audit access logs
- Use private registries

### 4. Performance
- Use parallelism wisely
- Implement caching
- Optimize provider calls
- Use data sources efficiently
- Minimize dependencies
- Profile execution time

---

## ✅ Completion Checklist

### Week 1
- [ ] Understand Terraform workflow
- [ ] Master HCL syntax
- [ ] Create resources and modules
- [ ] Manage state effectively
- [ ] Use variables and outputs

### Week 2
- [ ] Configure remote backends
- [ ] Implement workspaces
- [ ] Use dynamic blocks
- [ ] Integrate with CI/CD
- [ ] Complete infrastructure project

---

**Next:** Continue with specialized topics or return to [Main README](../../README.md)