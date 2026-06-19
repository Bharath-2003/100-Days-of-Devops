# AWS (Amazon Web Services) for DevOps

## 🎯 Overview

AWS is the world's most comprehensive cloud platform with over 200 services. For DevOps engineers, mastering core AWS services is essential for building scalable, reliable infrastructure.

**Duration:** 6 weeks (Weeks 29-34)
**Time Investment:** 4-5 hours/day
**Difficulty:** Intermediate to Advanced
**Prerequisites:** Linux, Networking, Basic cloud concepts

---

## 📚 What You'll Learn

### Week 1: AWS Fundamentals
- AWS account setup and IAM
- AWS CLI and SDK
- Regions and Availability Zones
- EC2 (Elastic Compute Cloud)
- EBS (Elastic Block Store)

### Week 2: Networking
- VPC (Virtual Private Cloud)
- Subnets and Route Tables
- Internet Gateway and NAT Gateway
- Security Groups and NACLs
- Elastic Load Balancer

### Week 3: Storage and Databases
- S3 (Simple Storage Service)
- RDS (Relational Database Service)
- DynamoDB
- ElastiCache
- EFS (Elastic File System)

### Week 4: Application Services
- Lambda (Serverless)
- API Gateway
- SQS and SNS
- ECS and EKS
- CloudWatch

### Week 5: Infrastructure as Code
- CloudFormation
- AWS CDK
- Terraform with AWS
- Systems Manager
- Secrets Manager

### Week 6: DevOps Services
- CodePipeline
- CodeBuild
- CodeDeploy
- ECR (Container Registry)
- Cost Optimization

---

## 🎓 Important Concepts

### 1. AWS Global Infrastructure

```
AWS Global Infrastructure
├─ Regions (26+)
│  ├─ US East (N. Virginia) - us-east-1
│  ├─ US West (Oregon) - us-west-2
│  ├─ EU (Ireland) - eu-west-1
│  ├─ Asia Pacific (Mumbai) - ap-south-1
│  └─ ...
│
├─ Availability Zones (84+)
│  ├─ us-east-1a
│  ├─ us-east-1b
│  ├─ us-east-1c
│  └─ ...
│
└─ Edge Locations (400+)
   └─ CloudFront CDN
```

**Key Concepts:**
- **Region:** Geographic area with multiple AZs
- **Availability Zone:** Isolated data center within region
- **Edge Location:** CDN endpoint for CloudFront
- **Local Zone:** Extension of region for low latency

**Best Practices:**
- Deploy across multiple AZs for high availability
- Choose region based on latency and compliance
- Use CloudFront for global content delivery

---

### 2. IAM (Identity and Access Management)

```
IAM Structure
├─ Users (Individual identities)
├─ Groups (Collection of users)
├─ Roles (Temporary credentials)
└─ Policies (Permissions)
    ├─ AWS Managed
    ├─ Customer Managed
    └─ Inline
```

**IAM Policy Example:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:Describe*",
        "ec2:StartInstances",
        "ec2:StopInstances"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "ec2:Region": "us-east-1"
        }
      }
    }
  ]
}
```

**IAM Best Practices:**
- Enable MFA for root account
- Use IAM roles for EC2 instances
- Follow principle of least privilege
- Rotate credentials regularly
- Use IAM Access Analyzer
- Enable CloudTrail for auditing

---

### 3. EC2 (Elastic Compute Cloud)

```
EC2 Instance Types
├─ General Purpose (t3, m5)
├─ Compute Optimized (c5, c6g)
├─ Memory Optimized (r5, x1)
├─ Storage Optimized (i3, d2)
└─ Accelerated Computing (p3, g4)

Instance Lifecycle
┌─────────┐
│ Pending │ ← Launch
└────┬────┘
     │
┌────▼────┐
│ Running │ ← Start/Stop
└────┬────┘
     │
┌────▼────┐
│ Stopped │
└────┬────┘
     │
┌────▼────────┐
│ Terminated  │
└─────────────┘
```

**EC2 Pricing Models:**
- **On-Demand:** Pay per hour/second
- **Reserved:** 1-3 year commitment (up to 75% savings)
- **Spot:** Bid for unused capacity (up to 90% savings)
- **Savings Plans:** Flexible pricing (up to 72% savings)
- **Dedicated Hosts:** Physical server

---

### 4. VPC (Virtual Private Cloud)

```
VPC Architecture
┌─────────────────────────────────────────┐
│ VPC (10.0.0.0/16)                       │
│                                         │
│  ┌──────────────────────────────────┐  │
│  │ Public Subnet (10.0.1.0/24)      │  │
│  │  ┌────────┐  ┌────────┐         │  │
│  │  │  EC2   │  │  NAT   │         │  │
│  │  │  Web   │  │Gateway │         │  │
│  │  └────────┘  └────────┘         │  │
│  └──────────────────────────────────┘  │
│                                         │
│  ┌──────────────────────────────────┐  │
│  │ Private Subnet (10.0.2.0/24)     │  │
│  │  ┌────────┐  ┌────────┐         │  │
│  │  │  EC2   │  │  RDS   │         │  │
│  │  │  App   │  │        │         │  │
│  │  └────────┘  └────────┘         │  │
│  └──────────────────────────────────┘  │
│                                         │
│  ┌──────────────────────────────────┐  │
│  │ Internet Gateway                  │  │
│  └──────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

**VPC Components:**
- **Subnets:** IP address ranges
- **Route Tables:** Traffic routing rules
- **Internet Gateway:** Internet access
- **NAT Gateway:** Outbound internet for private subnets
- **Security Groups:** Instance-level firewall
- **NACLs:** Subnet-level firewall

---

### 5. S3 (Simple Storage Service)

```
S3 Storage Classes
├─ S3 Standard (Frequent access)
├─ S3 Intelligent-Tiering (Auto-tiering)
├─ S3 Standard-IA (Infrequent access)
├─ S3 One Zone-IA (Single AZ)
├─ S3 Glacier Instant Retrieval
├─ S3 Glacier Flexible Retrieval
└─ S3 Glacier Deep Archive

S3 Bucket Structure
bucket-name/
├─ folder1/
│  ├─ file1.txt
│  └─ file2.jpg
├─ folder2/
│  └─ data.csv
└─ logs/
   └─ access.log
```

**S3 Features:**
- Versioning
- Lifecycle policies
- Replication (CRR, SRR)
- Encryption (SSE-S3, SSE-KMS, SSE-C)
- Access control (Bucket policies, ACLs)
- Static website hosting

---

### 6. Load Balancing

```
Load Balancer Types
├─ Application Load Balancer (ALB)
│  ├─ Layer 7 (HTTP/HTTPS)
│  ├─ Path-based routing
│  ├─ Host-based routing
│  └─ WebSocket support
│
├─ Network Load Balancer (NLB)
│  ├─ Layer 4 (TCP/UDP)
│  ├─ Ultra-low latency
│  ├─ Static IP
│  └─ Millions of requests/sec
│
└─ Classic Load Balancer (CLB)
   └─ Legacy (Layer 4 & 7)

ALB Architecture
┌──────────────────────┐
│  Application LB      │
├──────────────────────┤
│  Listener (80, 443)  │
└──────┬───────────────┘
       │
   ┌───┴────┐
   │ Rules  │
   └───┬────┘
       │
┌──────┴──────┐
│Target Groups│
├─────────────┤
│ ┌─────────┐ │
│ │  EC2-1  │ │
│ │  EC2-2  │ │
│ │  EC2-3  │ │
│ └─────────┘ │
└─────────────┘
```

---

### 7. RDS (Relational Database Service)

```
RDS Engines
├─ MySQL
├─ PostgreSQL
├─ MariaDB
├─ Oracle
├─ SQL Server
└─ Aurora (MySQL/PostgreSQL compatible)

RDS Deployment
┌─────────────────────────────┐
│ Primary DB (AZ-1)           │
│  ┌──────────────────────┐   │
│  │  RDS Instance        │   │
│  │  (Read/Write)        │   │
│  └──────────┬───────────┘   │
└─────────────┼───────────────┘
              │ Synchronous
              │ Replication
┌─────────────▼───────────────┐
│ Standby DB (AZ-2)           │
│  ┌──────────────────────┐   │
│  │  RDS Instance        │   │
│  │  (Standby)           │   │
│  └──────────────────────┘   │
└─────────────────────────────┘
```

**RDS Features:**
- Automated backups
- Multi-AZ deployment
- Read replicas
- Automated patching
- Encryption at rest
- Performance Insights

---

## 💻 Hands-On Exercises

### Exercise 1: AWS CLI Setup (30 minutes)

**Objective:** Configure AWS CLI and basic operations

```bash
# 1. Install AWS CLI
# macOS
brew install awscli

# Linux
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Windows
# Download and run installer from AWS website

# 2. Verify installation
aws --version

# 3. Configure AWS CLI
aws configure
# AWS Access Key ID: YOUR_ACCESS_KEY
# AWS Secret Access Key: YOUR_SECRET_KEY
# Default region: us-east-1
# Default output format: json

# 4. Test configuration
aws sts get-caller-identity

# 5. List S3 buckets
aws s3 ls

# 6. List EC2 instances
aws ec2 describe-instances

# 7. Get current region
aws configure get region

# 8. Use profiles
aws configure --profile dev
aws s3 ls --profile dev

# 9. Set default profile
export AWS_PROFILE=dev

# 10. Use different output formats
aws ec2 describe-instances --output table
aws ec2 describe-instances --output yaml
aws ec2 describe-instances --output text
```

**Challenge:**
Create a script that:
1. Lists all EC2 instances
2. Shows their state
3. Displays public IPs
4. Exports to CSV

**Solution:**
```bash
#!/bin/bash
aws ec2 describe-instances \
  --query 'Reservations[*].Instances[*].[InstanceId,State.Name,PublicIpAddress,Tags[?Key==`Name`].Value|[0]]' \
  --output text | \
  awk 'BEGIN {print "InstanceID,State,PublicIP,Name"} {print $1","$2","$3","$4}' > instances.csv
```

---

### Exercise 2: EC2 Instance Management (60 minutes)

**Objective:** Launch and manage EC2 instances

```bash
# 1. Create key pair
aws ec2 create-key-pair \
  --key-name my-key \
  --query 'KeyMaterial' \
  --output text > my-key.pem

chmod 400 my-key.pem

# 2. Create security group
aws ec2 create-security-group \
  --group-name web-sg \
  --description "Web server security group"

# Get security group ID
SG_ID=$(aws ec2 describe-security-groups \
  --group-names web-sg \
  --query 'SecurityGroups[0].GroupId' \
  --output text)

# 3. Add inbound rules
aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# 4. Launch EC2 instance
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t2.micro \
  --key-name my-key \
  --security-group-ids $SG_ID \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=WebServer}]' \
  --user-data file://user-data.sh

# 5. Get instance ID
INSTANCE_ID=$(aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=WebServer" \
  --query 'Reservations[0].Instances[0].InstanceId' \
  --output text)

# 6. Wait for instance to be running
aws ec2 wait instance-running --instance-ids $INSTANCE_ID

# 7. Get public IP
PUBLIC_IP=$(aws ec2 describe-instances \
  --instance-ids $INSTANCE_ID \
  --query 'Reservations[0].Instances[0].PublicIpAddress' \
  --output text)

echo "Instance is running at: $PUBLIC_IP"

# 8. Connect to instance
ssh -i my-key.pem ec2-user@$PUBLIC_IP

# 9. Stop instance
aws ec2 stop-instances --instance-ids $INSTANCE_ID

# 10. Start instance
aws ec2 start-instances --instance-ids $INSTANCE_ID

# 11. Terminate instance
aws ec2 terminate-instances --instance-ids $INSTANCE_ID
```

**User Data Script (user-data.sh):**
```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello from AWS EC2!</h1>" > /var/www/html/index.html
```

**Challenge:**
Create a script that:
1. Launches 3 EC2 instances
2. Tags them appropriately
3. Waits for all to be running
4. Displays their IPs
5. Installs Nginx on all

---

### Exercise 3: S3 Bucket Operations (45 minutes)

**Objective:** Work with S3 buckets and objects

```bash
# 1. Create bucket
aws s3 mb s3://my-devops-bucket-12345

# 2. List buckets
aws s3 ls

# 3. Upload file
echo "Hello S3" > test.txt
aws s3 cp test.txt s3://my-devops-bucket-12345/

# 4. List objects
aws s3 ls s3://my-devops-bucket-12345/

# 5. Download file
aws s3 cp s3://my-devops-bucket-12345/test.txt downloaded.txt

# 6. Sync directory
mkdir myfiles
echo "File 1" > myfiles/file1.txt
echo "File 2" > myfiles/file2.txt
aws s3 sync myfiles/ s3://my-devops-bucket-12345/myfiles/

# 7. Enable versioning
aws s3api put-bucket-versioning \
  --bucket my-devops-bucket-12345 \
  --versioning-configuration Status=Enabled

# 8. Set bucket policy
cat > bucket-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-devops-bucket-12345/*"
    }
  ]
}
EOF

aws s3api put-bucket-policy \
  --bucket my-devops-bucket-12345 \
  --policy file://bucket-policy.json

# 9. Enable static website hosting
aws s3 website s3://my-devops-bucket-12345/ \
  --index-document index.html \
  --error-document error.html

# 10. Set lifecycle policy
cat > lifecycle.json << 'EOF'
{
  "Rules": [
    {
      "Id": "Move to IA after 30 days",
      "Status": "Enabled",
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "STANDARD_IA"
        },
        {
          "Days": 90,
          "StorageClass": "GLACIER"
        }
      ]
    }
  ]
}
EOF

aws s3api put-bucket-lifecycle-configuration \
  --bucket my-devops-bucket-12345 \
  --lifecycle-configuration file://lifecycle.json

# 11. Delete objects
aws s3 rm s3://my-devops-bucket-12345/test.txt

# 12. Delete bucket
aws s3 rb s3://my-devops-bucket-12345 --force
```

**Challenge:**
Create a backup script that:
1. Compresses local directory
2. Uploads to S3 with timestamp
3. Keeps only last 7 backups
4. Sends SNS notification on completion

---

### Exercise 4: VPC Creation (90 minutes)

**Objective:** Create a complete VPC with public and private subnets

```bash
# 1. Create VPC
VPC_ID=$(aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=MyVPC}]' \
  --query 'Vpc.VpcId' \
  --output text)

# 2. Enable DNS hostnames
aws ec2 modify-vpc-attribute \
  --vpc-id $VPC_ID \
  --enable-dns-hostnames

# 3. Create Internet Gateway
IGW_ID=$(aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=MyIGW}]' \
  --query 'InternetGateway.InternetGatewayId' \
  --output text)

# 4. Attach IGW to VPC
aws ec2 attach-internet-gateway \
  --vpc-id $VPC_ID \
  --internet-gateway-id $IGW_ID

# 5. Create public subnet
PUBLIC_SUBNET_ID=$(aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=PublicSubnet}]' \
  --query 'Subnet.SubnetId' \
  --output text)

# 6. Create private subnet
PRIVATE_SUBNET_ID=$(aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.2.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=PrivateSubnet}]' \
  --query 'Subnet.SubnetId' \
  --output text)

# 7. Create public route table
PUBLIC_RT_ID=$(aws ec2 create-route-table \
  --vpc-id $VPC_ID \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=PublicRT}]' \
  --query 'RouteTable.RouteTableId' \
  --output text)

# 8. Add route to Internet Gateway
aws ec2 create-route \
  --route-table-id $PUBLIC_RT_ID \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id $IGW_ID

# 9. Associate public subnet with route table
aws ec2 associate-route-table \
  --subnet-id $PUBLIC_SUBNET_ID \
  --route-table-id $PUBLIC_RT_ID

# 10. Allocate Elastic IP for NAT Gateway
EIP_ALLOC_ID=$(aws ec2 allocate-address \
  --domain vpc \
  --query 'AllocationId' \
  --output text)

# 11. Create NAT Gateway
NAT_GW_ID=$(aws ec2 create-nat-gateway \
  --subnet-id $PUBLIC_SUBNET_ID \
  --allocation-id $EIP_ALLOC_ID \
  --tag-specifications 'ResourceType=natgateway,Tags=[{Key=Name,Value=MyNATGW}]' \
  --query 'NatGateway.NatGatewayId' \
  --output text)

# Wait for NAT Gateway to be available
aws ec2 wait nat-gateway-available --nat-gateway-ids $NAT_GW_ID

# 12. Create private route table
PRIVATE_RT_ID=$(aws ec2 create-route-table \
  --vpc-id $VPC_ID \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=PrivateRT}]' \
  --query 'RouteTable.RouteTableId' \
  --output text)

# 13. Add route to NAT Gateway
aws ec2 create-route \
  --route-table-id $PRIVATE_RT_ID \
  --destination-cidr-block 0.0.0.0/0 \
  --nat-gateway-id $NAT_GW_ID

# 14. Associate private subnet with route table
aws ec2 associate-route-table \
  --subnet-id $PRIVATE_SUBNET_ID \
  --route-table-id $PRIVATE_RT_ID

echo "VPC Setup Complete!"
echo "VPC ID: $VPC_ID"
echo "Public Subnet: $PUBLIC_SUBNET_ID"
echo "Private Subnet: $PRIVATE_SUBNET_ID"
```

**Challenge:**
Extend the VPC to:
1. Add second AZ for high availability
2. Create additional public/private subnets
3. Set up VPC peering
4. Configure VPC Flow Logs
5. Add VPC endpoints for S3

---

### Exercise 5: Application Load Balancer (60 minutes)

**Objective:** Set up ALB with target groups

```bash
# 1. Create target group
TG_ARN=$(aws elbv2 create-target-group \
  --name web-targets \
  --protocol HTTP \
  --port 80 \
  --vpc-id $VPC_ID \
  --health-check-path /health \
  --health-check-interval-seconds 30 \
  --health-check-timeout-seconds 5 \
  --healthy-threshold-count 2 \
  --unhealthy-threshold-count 3 \
  --query 'TargetGroups[0].TargetGroupArn' \
  --output text)

# 2. Create Application Load Balancer
ALB_ARN=$(aws elbv2 create-load-balancer \
  --name my-alb \
  --subnets $PUBLIC_SUBNET_ID $PUBLIC_SUBNET_ID2 \
  --security-groups $ALB_SG_ID \
  --scheme internet-facing \
  --type application \
  --ip-address-type ipv4 \
  --query 'LoadBalancers[0].LoadBalancerArn' \
  --output text)

# 3. Create listener
LISTENER_ARN=$(aws elbv2 create-listener \
  --load-balancer-arn $ALB_ARN \
  --protocol HTTP \
  --port 80 \
  --default-actions Type=forward,TargetGroupArn=$TG_ARN \
  --query 'Listeners[0].ListenerArn' \
  --output text)

# 4. Register targets
aws elbv2 register-targets \
  --target-group-arn $TG_ARN \
  --targets Id=$INSTANCE_ID1 Id=$INSTANCE_ID2

# 5. Get ALB DNS name
ALB_DNS=$(aws elbv2 describe-load-balancers \
  --load-balancer-arns $ALB_ARN \
  --query 'LoadBalancers[0].DNSName' \
  --output text)

echo "ALB DNS: $ALB_DNS"

# 6. Test ALB
curl http://$ALB_DNS

# 7. Add path-based routing
aws elbv2 create-rule \
  --listener-arn $LISTENER_ARN \
  --priority 10 \
  --conditions Field=path-pattern,Values='/api/*' \
  --actions Type=forward,TargetGroupArn=$API_TG_ARN
```

---

### Exercise 6: RDS Database (45 minutes)

**Objective:** Create and manage RDS instance

```bash
# 1. Create DB subnet group
aws rds create-db-subnet-group \
  --db-subnet-group-name my-db-subnet-group \
  --db-subnet-group-description "My DB subnet group" \
  --subnet-ids $PRIVATE_SUBNET_ID1 $PRIVATE_SUBNET_ID2

# 2. Create security group for RDS
DB_SG_ID=$(aws ec2 create-security-group \
  --group-name db-sg \
  --description "Database security group" \
  --vpc-id $VPC_ID \
  --query 'GroupId' \
  --output text)

# 3. Allow MySQL access from app servers
aws ec2 authorize-security-group-ingress \
  --group-id $DB_SG_ID \
  --protocol tcp \
  --port 3306 \
  --source-group $APP_SG_ID

# 4. Create RDS instance
aws rds create-db-instance \
  --db-instance-identifier mydb \
  --db-instance-class db.t3.micro \
  --engine mysql \
  --master-username admin \
  --master-user-password MySecurePass123 \
  --allocated-storage 20 \
  --vpc-security-group-ids $DB_SG_ID \
  --db-subnet-group-name my-db-subnet-group \
  --backup-retention-period 7 \
  --preferred-backup-window "03:00-04:00" \
  --preferred-maintenance-window "mon:04:00-mon:05:00" \
  --multi-az \
  --storage-encrypted \
  --enable-cloudwatch-logs-exports '["error","general","slowquery"]'

# 5. Wait for DB to be available
aws rds wait db-instance-available --db-instance-identifier mydb

# 6. Get endpoint
DB_ENDPOINT=$(aws rds describe-db-instances \
  --db-instance-identifier mydb \
  --query 'DBInstances[0].Endpoint.Address' \
  --output text)

echo "Database endpoint: $DB_ENDPOINT"

# 7. Create read replica
aws rds create-db-instance-read-replica \
  --db-instance-identifier mydb-replica \
  --source-db-instance-identifier mydb \
  --db-instance-class db.t3.micro

# 8. Create snapshot
aws rds create-db-snapshot \
  --db-instance-identifier mydb \
  --db-snapshot-identifier mydb-snapshot-$(date +%Y%m%d)

# 9. Modify instance
aws rds modify-db-instance \
  --db-instance-identifier mydb \
  --allocated-storage 30 \
  --apply-immediately
```

---

### Exercise 7: Lambda Function (45 minutes)

**Objective:** Create and deploy serverless function

```python
# lambda_function.py
import json
import boto3

def lambda_handler(event, context):
    # Get S3 client
    s3 = boto3.client('s3')
    
    # List buckets
    response = s3.list_buckets()
    buckets = [bucket['Name'] for bucket in response['Buckets']]
    
    return {
        'statusCode': 200,
        'body': json.dumps({
            'message': 'Hello from Lambda!',
            'buckets': buckets
        })
    }
```

```bash
# 1. Create IAM role for Lambda
cat > trust-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

ROLE_ARN=$(aws iam create-role \
  --role-name lambda-execution-role \
  --assume-role-policy-document file://trust-policy.json \
  --query 'Role.Arn' \
  --output text)

# 2. Attach policies
aws iam attach-role-policy \
  --role-name lambda-execution-role \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

aws iam attach-role-policy \
  --role-name lambda-execution-role \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# 3. Package function
zip function.zip lambda_function.py

# 4. Create Lambda function
aws lambda create-function \
  --function-name my-function \
  --runtime python3.9 \
  --role $ROLE_ARN \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip \
  --timeout 30 \
  --memory-size 128

# 5. Invoke function
aws lambda invoke \
  --function-name my-function \
  --payload '{}' \
  response.json

cat response.json

# 6. Update function code
aws lambda update-function-code \
  --function-name my-function \
  --zip-file fileb://function.zip

# 7. Add environment variables
aws lambda update-function-configuration \
  --function-name my-function \
  --environment Variables={BUCKET_NAME=my-bucket}

# 8. Create API Gateway trigger
# (Use AWS Console or CloudFormation for complex setup)
```

---

## 🎯 Practice Projects

### Project 1: Three-Tier Web Application
Deploy a complete web application:
- ALB in public subnets
- EC2 instances in private subnets
- RDS in database subnets
- Auto Scaling Group
- CloudWatch monitoring
- S3 for static assets
- CloudFront CDN

### Project 2: CI/CD Pipeline
Build complete CI/CD:
- CodeCommit for source control
- CodeBuild for building
- CodeDeploy for deployment
- CodePipeline for orchestration
- ECR for Docker images
- ECS for container hosting

### Project 3: Serverless Application
Create serverless app:
- API Gateway
- Lambda functions
- DynamoDB
- S3 for storage
- CloudWatch for monitoring
- Cognito for authentication

### Project 4: Disaster Recovery
Implement DR solution:
- Multi-region setup
- RDS cross-region replication
- S3 cross-region replication
- Route53 failover
- Automated backups
- Recovery testing

---

## 📝 Best Practices

### 1. Security
- Enable MFA on root account
- Use IAM roles, not access keys
- Encrypt data at rest and in transit
- Use VPC for network isolation
- Enable CloudTrail for auditing
- Regular security audits
- Use AWS Security Hub

### 2. Cost Optimization
- Use Reserved Instances for steady workloads
- Use Spot Instances for flexible workloads
- Right-size instances
- Use S3 lifecycle policies
- Delete unused resources
- Use AWS Cost Explorer
- Set up billing alerts

### 3. High Availability
- Deploy across multiple AZs
- Use Auto Scaling
- Use Load Balancers
- Enable RDS Multi-AZ
- Use Route53 health checks
- Implement graceful degradation

### 4. Monitoring
- Enable CloudWatch metrics
- Set up CloudWatch alarms
- Use CloudWatch Logs
- Enable X-Ray for tracing
- Use CloudWatch Dashboards
- Set up SNS notifications

---

## ✅ Completion Checklist

### Week 1
- [ ] Set up AWS account and IAM
- [ ] Configure AWS CLI
- [ ] Launch and manage EC2 instances
- [ ] Work with EBS volumes
- [ ] Understand pricing models

### Week 2
- [ ] Create VPC with subnets
- [ ] Configure routing and gateways
- [ ] Set up security groups
- [ ] Deploy Load Balancer
- [ ] Understand networking concepts

### Week 3
- [ ] Work with S3 buckets
- [ ] Create RDS instances
- [ ] Use DynamoDB
- [ ] Configure ElastiCache
- [ ] Implement backup strategies

### Week 4
- [ ] Create Lambda functions
- [ ] Set up API Gateway
- [ ] Use SQS and SNS
- [ ] Deploy to ECS/EKS
- [ ] Configure CloudWatch

### Week 5
- [ ] Write CloudFormation templates
- [ ] Use AWS CDK
- [ ] Integrate Terraform
- [ ] Use Systems Manager
- [ ] Manage secrets

### Week 6
- [ ] Build CI/CD pipeline
- [ ] Use CodePipeline
- [ ] Deploy with CodeDeploy
- [ ] Optimize costs
- [ ] Complete projects

---

**Next:** [Azure →](../azure/README.md)

**Last Updated:** June 19, 2026