# Project 7: Cloud Resume Challenge

## 🎯 Project Overview

**Difficulty:** Advanced (Capstone Project)  
**Duration:** 12-16 hours  
**Technologies:** AWS, Terraform, GitHub Actions, S3, CloudFront, Lambda, DynamoDB, API Gateway, Route53

### What You'll Build
A complete serverless resume website hosted on AWS with visitor counter, CI/CD pipeline, infrastructure as code, and full DevOps automation - demonstrating end-to-end cloud and DevOps skills.

### Learning Objectives
- ✅ Build serverless architecture on AWS
- ✅ Implement Infrastructure as Code
- ✅ Create CI/CD pipeline
- ✅ Configure CDN and SSL
- ✅ Build REST API with Lambda
- ✅ Implement database integration
- ✅ Set up custom domain
- ✅ Automate deployments

### Prerequisites
- AWS account
- Domain name (optional but recommended)
- GitHub account
- Terraform installed
- AWS CLI configured
- All previous projects completed

---

## 📋 Project Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         Internet                                 │
└─────────────────────────────────────────────────────────────────┘
                           │
                           │ HTTPS
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Route 53 (DNS)                                │
│              resume.yourdomain.com                               │
└─────────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│              CloudFront (CDN + SSL)                              │
│  ┌────────────────────────────────────────────────────────┐    │
│  │  - Global Edge Locations                                │    │
│  │  - SSL/TLS Certificate (ACM)                            │    │
│  │  - Caching                                              │    │
│  │  - HTTPS Redirect                                       │    │
│  └────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                    S3 Bucket (Static Website)                    │
│  ┌────────────────────────────────────────────────────────┐    │
│  │  - index.html                                           │    │
│  │  - styles.css                                           │    │
│  │  - script.js                                            │    │
│  │  - resume.pdf                                           │    │
│  └────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
                           │
                           │ API Call
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│              API Gateway (REST API)                              │
│  ┌────────────────────────────────────────────────────────┐    │
│  │  GET /visitor-count                                     │    │
│  │  POST /visitor-count                                    │    │
│  │  - CORS Enabled                                         │    │
│  │  - API Key (optional)                                   │    │
│  └────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│              Lambda Function (Python/Node.js)                    │
│  ┌────────────────────────────────────────────────────────┐    │
│  │  - Get visitor count                                    │    │
│  │  - Increment counter                                    │    │
│  │  - Return JSON response                                 │    │
│  └────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│              DynamoDB (NoSQL Database)                           │
│  ┌────────────────────────────────────────────────────────┐    │
│  │  Table: visitor-counter                                 │    │
│  │  - id: "main"                                           │    │
│  │  - count: <number>                                      │    │
│  └────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    GitHub Actions CI/CD                          │
│  ┌────────────────────────────────────────────────────────┐    │
│  │  Frontend Pipeline:                                     │    │
│  │  - Build & Test                                         │    │
│  │  - Deploy to S3                                         │    │
│  │  - Invalidate CloudFront                                │    │
│  │                                                          │    │
│  │  Backend Pipeline:                                      │    │
│  │  - Test Lambda                                          │    │
│  │  - Deploy Lambda                                        │    │
│  │  - Update API Gateway                                   │    │
│  │                                                          │    │
│  │  Infrastructure Pipeline:                               │    │
│  │  - Terraform Plan                                       │    │
│  │  - Terraform Apply                                      │    │
│  └────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🚀 Step-by-Step Implementation

### Phase 1: Frontend Development

#### Step 1.1: Create Project Structure

```bash
mkdir -p cloud-resume-challenge
cd cloud-resume-challenge

# Create directory structure
mkdir -p {frontend/{src,dist},backend/{lambda,tests},infrastructure,tests,.github/workflows}

# Create files
touch frontend/src/{index.html,styles.css,script.js}
touch backend/lambda/visitor_counter.py
touch infrastructure/{main.tf,variables.tf,outputs.tf}
```

#### Step 1.2: Create HTML Resume

**File: `frontend/src/index.html`**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>John Doe - DevOps Engineer</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <div class="container">
        <header>
            <h1>John Doe</h1>
            <p class="title">DevOps Engineer | Cloud Architect</p>
            <div class="contact">
                <a href="mailto:john@example.com">john@example.com</a> |
                <a href="https://linkedin.com/in/johndoe">LinkedIn</a> |
                <a href="https://github.com/johndoe">GitHub</a>
            </div>
        </header>

        <section class="summary">
            <h2>Professional Summary</h2>
            <p>
                Experienced DevOps Engineer with 5+ years of expertise in cloud infrastructure,
                CI/CD pipelines, and container orchestration. Proven track record of implementing
                scalable solutions and automating deployment processes.
            </p>
        </section>

        <section class="skills">
            <h2>Technical Skills</h2>
            <div class="skill-grid">
                <div class="skill-category">
                    <h3>Cloud Platforms</h3>
                    <ul>
                        <li>AWS (EC2, S3, Lambda, ECS, EKS)</li>
                        <li>Azure (VMs, AKS, Functions)</li>
                        <li>GCP (Compute Engine, GKE)</li>
                    </ul>
                </div>
                <div class="skill-category">
                    <h3>DevOps Tools</h3>
                    <ul>
                        <li>Docker & Kubernetes</li>
                        <li>Terraform & Ansible</li>
                        <li>Jenkins & GitHub Actions</li>
                        <li>ArgoCD & GitOps</li>
                    </ul>
                </div>
                <div class="skill-category">
                    <h3>Monitoring</h3>
                    <ul>
                        <li>Prometheus & Grafana</li>
                        <li>ELK Stack</li>
                        <li>CloudWatch</li>
                    </ul>
                </div>
                <div class="skill-category">
                    <h3>Programming</h3>
                    <ul>
                        <li>Python, Bash, Go</li>
                        <li>JavaScript/Node.js</li>
                        <li>SQL & NoSQL</li>
                    </ul>
                </div>
            </div>
        </section>

        <section class="experience">
            <h2>Professional Experience</h2>
            
            <div class="job">
                <h3>Senior DevOps Engineer</h3>
                <p class="company">Tech Company Inc. | 2021 - Present</p>
                <ul>
                    <li>Designed and implemented AWS infrastructure using Terraform</li>
                    <li>Built CI/CD pipelines reducing deployment time by 60%</li>
                    <li>Managed Kubernetes clusters serving 1M+ daily users</li>
                    <li>Implemented monitoring stack with Prometheus and Grafana</li>
                </ul>
            </div>

            <div class="job">
                <h3>DevOps Engineer</h3>
                <p class="company">Startup Co. | 2019 - 2021</p>
                <ul>
                    <li>Migrated legacy applications to containerized architecture</li>
                    <li>Automated infrastructure provisioning with Ansible</li>
                    <li>Implemented GitOps workflow with ArgoCD</li>
                    <li>Reduced infrastructure costs by 40%</li>
                </ul>
            </div>
        </section>

        <section class="certifications">
            <h2>Certifications</h2>
            <ul>
                <li>AWS Certified Solutions Architect - Professional</li>
                <li>Certified Kubernetes Administrator (CKA)</li>
                <li>HashiCorp Certified: Terraform Associate</li>
                <li>AWS Certified DevOps Engineer - Professional</li>
            </ul>
        </section>

        <section class="projects">
            <h2>Notable Projects</h2>
            
            <div class="project">
                <h3>Multi-Cloud Infrastructure Platform</h3>
                <p>Built unified infrastructure management platform supporting AWS, Azure, and GCP</p>
                <p class="tech">Technologies: Terraform, Python, Kubernetes, ArgoCD</p>
            </div>

            <div class="project">
                <h3>Automated CI/CD Pipeline</h3>
                <p>Designed end-to-end CI/CD pipeline with automated testing and deployment</p>
                <p class="tech">Technologies: GitHub Actions, Docker, Kubernetes, Helm</p>
            </div>
        </section>

        <footer>
            <div class="visitor-counter">
                <p>Visitor Count: <span id="visitor-count">Loading...</span></p>
            </div>
            <p>&copy; 2024 John Doe. Built with AWS, Terraform, and GitHub Actions.</p>
        </footer>
    </div>

    <script src="script.js"></script>
</body>
</html>
```

#### Step 1.3: Create CSS Styling

**File: `frontend/src/styles.css`**
```css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    line-height: 1.6;
    color: #333;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    padding: 20px;
}

.container {
    max-width: 900px;
    margin: 0 auto;
    background: white;
    padding: 40px;
    border-radius: 10px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
}

header {
    text-align: center;
    padding-bottom: 30px;
    border-bottom: 3px solid #667eea;
    margin-bottom: 30px;
}

h1 {
    font-size: 2.5em;
    color: #667eea;
    margin-bottom: 10px;
}

.title {
    font-size: 1.2em;
    color: #666;
    margin-bottom: 15px;
}

.contact a {
    color: #667eea;
    text-decoration: none;
    margin: 0 10px;
}

.contact a:hover {
    text-decoration: underline;
}

section {
    margin-bottom: 30px;
}

h2 {
    color: #667eea;
    font-size: 1.8em;
    margin-bottom: 15px;
    padding-bottom: 10px;
    border-bottom: 2px solid #eee;
}

h3 {
    color: #764ba2;
    font-size: 1.3em;
    margin-bottom: 10px;
}

.skill-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 20px;
    margin-top: 20px;
}

.skill-category {
    background: #f8f9fa;
    padding: 20px;
    border-radius: 8px;
    border-left: 4px solid #667eea;
}

.skill-category ul {
    list-style: none;
    padding-left: 0;
}

.skill-category li {
    padding: 5px 0;
    color: #555;
}

.skill-category li:before {
    content: "▹ ";
    color: #667eea;
    font-weight: bold;
}

.job, .project {
    margin-bottom: 25px;
    padding: 20px;
    background: #f8f9fa;
    border-radius: 8px;
    border-left: 4px solid #764ba2;
}

.company {
    color: #666;
    font-style: italic;
    margin-bottom: 10px;
}

.job ul, .certifications ul {
    margin-left: 20px;
    margin-top: 10px;
}

.job li, .certifications li {
    margin-bottom: 8px;
    color: #555;
}

.tech {
    color: #667eea;
    font-size: 0.9em;
    margin-top: 10px;
}

footer {
    text-align: center;
    padding-top: 30px;
    border-top: 2px solid #eee;
    color: #666;
}

.visitor-counter {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    padding: 15px;
    border-radius: 8px;
    margin-bottom: 20px;
    font-size: 1.1em;
}

#visitor-count {
    font-weight: bold;
    font-size: 1.3em;
}

@media (max-width: 768px) {
    .container {
        padding: 20px;
    }
    
    h1 {
        font-size: 2em;
    }
    
    .skill-grid {
        grid-template-columns: 1fr;
    }
}
```

#### Step 1.4: Create JavaScript for API Integration

**File: `frontend/src/script.js`**
```javascript
// API Gateway endpoint (will be replaced by Terraform output)
const API_ENDPOINT = 'https://your-api-id.execute-api.us-east-1.amazonaws.com/prod/visitor-count';

// Fetch and update visitor count
async function updateVisitorCount() {
    try {
        const response = await fetch(API_ENDPOINT, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
        });

        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }

        const data = await response.json();
        document.getElementById('visitor-count').textContent = data.count;
    } catch (error) {
        console.error('Error fetching visitor count:', error);
        document.getElementById('visitor-count').textContent = 'Error loading count';
    }
}

// Call function when page loads
document.addEventListener('DOMContentLoaded', updateVisitorCount);
```

---

### Phase 2: Backend Development

#### Step 2.1: Create Lambda Function

**File: `backend/lambda/visitor_counter.py`**
```python
import json
import boto3
from decimal import Decimal

# Initialize DynamoDB client
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('visitor-counter')

def lambda_handler(event, context):
    """
    Lambda function to track and return visitor count
    """
    try:
        # Update visitor count
        response = table.update_item(
            Key={'id': 'main'},
            UpdateExpression='ADD visitor_count :inc',
            ExpressionAttributeValues={':inc': 1},
            ReturnValues='UPDATED_NEW'
        )
        
        # Get updated count
        count = int(response['Attributes']['visitor_count'])
        
        # Return response
        return {
            'statusCode': 200,
            'headers': {
                'Content-Type': 'application/json',
                'Access-Control-Allow-Origin': '*',
                'Access-Control-Allow-Methods': 'GET,POST,OPTIONS',
                'Access-Control-Allow-Headers': 'Content-Type'
            },
            'body': json.dumps({
                'count': count,
                'message': 'Visitor count updated successfully'
            })
        }
    
    except Exception as e:
        print(f"Error: {str(e)}")
        return {
            'statusCode': 500,
            'headers': {
                'Content-Type': 'application/json',
                'Access-Control-Allow-Origin': '*'
            },
            'body': json.dumps({
                'error': 'Internal server error',
                'message': str(e)
            })
        }
```

#### Step 2.2: Create Lambda Tests

**File: `backend/tests/test_visitor_counter.py`**
```python
import json
import pytest
from moto import mock_dynamodb
import boto3
from lambda_function import lambda_handler

@mock_dynamodb
def test_visitor_counter():
    # Create mock DynamoDB table
    dynamodb = boto3.resource('dynamodb', region_name='us-east-1')
    table = dynamodb.create_table(
        TableName='visitor-counter',
        KeySchema=[{'AttributeName': 'id', 'KeyType': 'HASH'}],
        AttributeDefinitions=[{'AttributeName': 'id', 'AttributeType': 'S'}],
        BillingMode='PAY_PER_REQUEST'
    )
    
    # Initialize counter
    table.put_item(Item={'id': 'main', 'visitor_count': 0})
    
    # Test lambda function
    event = {}
    context = {}
    response = lambda_handler(event, context)
    
    # Assertions
    assert response['statusCode'] == 200
    body = json.loads(response['body'])
    assert 'count' in body
    assert body['count'] == 1
```

---

### Phase 3: Infrastructure as Code

#### Step 3.1: Main Terraform Configuration

**File: `infrastructure/main.tf`**
```hcl
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  
  backend "s3" {
    bucket = "your-terraform-state-bucket"
    key    = "cloud-resume/terraform.tfstate"
    region = "us-east-1"
  }
}

provider "aws" {
  region = var.aws_region
}

# S3 Bucket for website
resource "aws_s3_bucket" "website" {
  bucket = var.domain_name
}

resource "aws_s3_bucket_website_configuration" "website" {
  bucket = aws_s3_bucket.website.id
  
  index_document {
    suffix = "index.html"
  }
  
  error_document {
    key = "error.html"
  }
}

resource "aws_s3_bucket_public_access_block" "website" {
  bucket = aws_s3_bucket.website.id
  
  block_public_acls       = false
  block_public_policy     = false
  ignore_public_acls      = false
  restrict_public_buckets = false
}

resource "aws_s3_bucket_policy" "website" {
  bucket = aws_s3_bucket.website.id
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid       = "PublicReadGetObject"
        Effect    = "Allow"
        Principal = "*"
        Action    = "s3:GetObject"
        Resource  = "${aws_s3_bucket.website.arn}/*"
      }
    ]
  })
}

# DynamoDB Table
resource "aws_dynamodb_table" "visitor_counter" {
  name           = "visitor-counter"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "id"
  
  attribute {
    name = "id"
    type = "S"
  }
}

# Lambda Function
resource "aws_lambda_function" "visitor_counter" {
  filename      = "../backend/lambda/visitor_counter.zip"
  function_name = "visitor-counter"
  role          = aws_iam_role.lambda_role.arn
  handler       = "visitor_counter.lambda_handler"
  runtime       = "python3.11"
  
  environment {
    variables = {
      TABLE_NAME = aws_dynamodb_table.visitor_counter.name
    }
  }
}

# IAM Role for Lambda
resource "aws_iam_role" "lambda_role" {
  name = "visitor-counter-lambda-role"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "lambda.amazonaws.com"
        }
      }
    ]
  })
}

# API Gateway
resource "aws_apigatewayv2_api" "visitor_api" {
  name          = "visitor-counter-api"
  protocol_type = "HTTP"
  
  cors_configuration {
    allow_origins = ["*"]
    allow_methods = ["GET", "POST", "OPTIONS"]
    allow_headers = ["Content-Type"]
  }
}

# CloudFront Distribution
resource "aws_cloudfront_distribution" "website" {
  enabled             = true
  default_root_object = "index.html"
  aliases             = [var.domain_name]
  
  origin {
    domain_name = aws_s3_bucket.website.bucket_regional_domain_name
    origin_id   = "S3-${var.domain_name}"
  }
  
  default_cache_behavior {
    allowed_methods        = ["GET", "HEAD"]
    cached_methods         = ["GET", "HEAD"]
    target_origin_id       = "S3-${var.domain_name}"
    viewer_protocol_policy = "redirect-to-https"
    
    forwarded_values {
      query_string = false
      cookies {
        forward = "none"
      }
    }
  }
  
  viewer_certificate {
    acm_certificate_arn = var.certificate_arn
    ssl_support_method  = "sni-only"
  }
  
  restrictions {
    geo_restriction {
      restriction_type = "none"
    }
  }
}
```

---

### Phase 4: CI/CD Pipeline

**File: `.github/workflows/deploy.yml`**
```yaml
name: Deploy Cloud Resume

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test-backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: |
          cd backend
          pip install -r requirements.txt
          pip install pytest moto
      
      - name: Run tests
        run: |
          cd backend
          pytest tests/

  deploy-infrastructure:
    runs-on: ubuntu-latest
    needs: test-backend
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2
      
      - name: Terraform Init
        run: |
          cd infrastructure
          terraform init
      
      - name: Terraform Plan
        run: |
          cd infrastructure
          terraform plan
      
      - name: Terraform Apply
        run: |
          cd infrastructure
          terraform apply -auto-approve

  deploy-frontend:
    runs-on: ubuntu-latest
    needs: deploy-infrastructure
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Sync to S3
        run: |
          aws s3 sync frontend/src/ s3://your-bucket-name/ --delete
      
      - name: Invalidate CloudFront
        run: |
          aws cloudfront create-invalidation \
            --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} \
            --paths "/*"
```

---

## 🎯 Success Criteria

- [ ] Resume website live and accessible
- [ ] Custom domain configured
- [ ] HTTPS enabled via CloudFront
- [ ] Visitor counter working
- [ ] Lambda function deployed
- [ ] DynamoDB storing data
- [ ] CI/CD pipeline functional
- [ ] Infrastructure managed by Terraform

---

## 📚 Additional Resources

- [Cloud Resume Challenge](https://cloudresumechallenge.dev/)
- [AWS Serverless](https://aws.amazon.com/serverless/)
- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)

---

## 🎓 What You Learned

✅ Serverless architecture  
✅ Infrastructure as Code  
✅ CI/CD automation  
✅ AWS services integration  
✅ Frontend development  
✅ Backend API development  
✅ DNS and CDN configuration  
✅ End-to-end DevOps workflow  

---

**Project Status:** ✅ Complete  
**Last Updated:** 2024-01-16