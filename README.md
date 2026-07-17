# AWS Terraform Infrastructure

A modular Terraform project for provisioning AWS infrastructure with networking, compute, load balancing, database, object storage, and IAM resources.

This repository demonstrates infrastructure-as-code practices using reusable Terraform modules and an environment-based structure.

## What It Provisions

The `dev` environment creates:

- VPC with DNS support
- Public and private subnets
- Internet gateway and public routing
- EC2 instance
- Application Load Balancer
- ALB target group and HTTP listener
- PostgreSQL RDS instance
- S3 bucket with versioning
- IAM user, group, and policy attachment

## Architecture

```text
Internet
   |
   v
Application Load Balancer
   |
   v
EC2 Instance
   |
   v
PostgreSQL RDS

S3 and IAM resources are provisioned alongside the application infrastructure.
```

## Repository Structure

```text
.
├── envs/
│   └── dev/
│       ├── main.tf
│       ├── outputs.tf
│       └── provider.tf
├── modules/
│   ├── alb/
│   ├── ec2/
│   ├── iam/
│   ├── rds/
│   ├── s3/
│   └── vpc/
└── README.md
```

## Prerequisites

Install and configure:

- Terraform `>= 1.5.0`
- AWS CLI
- Git
- AWS account with permissions for VPC, EC2, ALB, RDS, S3, and IAM

Configure AWS credentials:

```bash
aws configure
```

Or export credentials:

```bash
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_DEFAULT_REGION="us-east-1"
```

PowerShell:

```powershell
$env:AWS_ACCESS_KEY_ID="your-access-key"
$env:AWS_SECRET_ACCESS_KEY="your-secret-key"
$env:AWS_DEFAULT_REGION="us-east-1"
```

## Usage

Clone the repository:

```bash
git clone https://github.com/bader2424/aws-terraform-infrastructure.git
cd aws-terraform-infrastructure/envs/dev
```

Initialize Terraform:

```bash
terraform init
```

Format and validate:

```bash
terraform fmt -recursive ../..
terraform validate
```

Preview changes:

```bash
terraform plan
```

Apply infrastructure:

```bash
terraform apply
```

View outputs:

```bash
terraform output
```

Destroy resources when finished:

```bash
terraform destroy
```

## Modules

| Module | Purpose |
|---|---|
| `vpc` | Creates networking foundation with VPC, subnets, gateway, and routing |
| `ec2` | Provisions compute instance and security group |
| `alb` | Creates public load balancer, target group, listener, and target attachment |
| `rds` | Provisions PostgreSQL database and subnet group |
| `s3` | Creates versioned object storage bucket |
| `iam` | Creates IAM user, group, and policy attachment |

## Key Infrastructure Concepts

This project demonstrates:

- Modular Terraform design
- Environment-based infrastructure structure
- AWS networking with public/private subnets
- Load-balanced application entry point
- Private database placement
- Object storage provisioning
- IAM resource management
- Terraform validation, planning, apply, and destroy workflow

## Configuration Notes

Before applying in your own AWS account, review and update:

- AWS region
- EC2 AMI ID
- EC2 instance type
- RDS username and password
- S3 bucket name
- IAM permissions
- Security group ingress rules
- Terraform backend/state strategy

## Author

Bader Bahashwan  
GitHub: https://github.com/bader2424