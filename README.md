# AWS Terraform Infrastructure

This repository contains a modular Terraform project for provisioning a demo AWS backend infrastructure. It uses separate modules for networking, compute, load balancing, database, object storage, and IAM, with a deployable `dev` environment under `envs/dev`.

The project is best treated as a learning or portfolio infrastructure-as-code example. Several values are intentionally simplified for demonstration and should be changed before using this in a real AWS account.

## What This Project Creates

The `dev` environment provisions:

- A VPC with DNS support enabled.
- Two public subnets.
- Two private subnets.
- An internet gateway and public route table.
- One EC2 instance in a private subnet.
- An Application Load Balancer in the public subnets.
- An ALB target group and listener on port `80`.
- A PostgreSQL RDS instance in private subnets.
- An S3 bucket with versioning enabled.
- An IAM user, group, and policy attachment.

## Architecture

```text
Internet
   |
   v
Application Load Balancer
   |
   v
EC2 instance in private subnet
   |
   +--> RDS PostgreSQL in private subnets

S3 bucket and IAM resources are provisioned alongside the application stack.
```

## Repository Structure

```text
.
|-- envs/
|   `-- dev/
|       |-- main.tf          # Wires all modules together for the dev environment
|       |-- outputs.tf       # Prints the main infrastructure outputs
|       `-- provider.tf      # AWS provider configuration for dev
|-- modules/
|   |-- alb/                 # Application Load Balancer, target group, listener
|   |-- ec2/                 # EC2 instance and app security group
|   |-- iam/                 # IAM user, group, and policy attachment
|   |-- rds/                 # PostgreSQL database and DB subnet group
|   |-- s3/                  # S3 bucket and bucket versioning
|   `-- vpc/                 # VPC, subnets, internet gateway, route table
|-- .gitignore
|-- .terraform.lock.hcl      # Root lock file; see cleanup notes below
|-- provider.tf              # Root provider file; see cleanup notes below
|-- variables.tf             # Root variables file; see cleanup notes below
`-- README.md
```

## Prerequisites

Install and configure these tools before running the project:

- [Git](https://git-scm.com/)
- [Terraform](https://developer.hashicorp.com/terraform/install) `>= 1.5.0`
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
- An AWS account with permissions to create VPC, EC2, ALB, RDS, S3, and IAM resources.

Configure AWS credentials with one of the standard AWS methods:

```bash
aws configure
```

Or export credentials in your shell:

```bash
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_DEFAULT_REGION="us-east-1"
```

For PowerShell:

```powershell
$env:AWS_ACCESS_KEY_ID="your-access-key"
$env:AWS_SECRET_ACCESS_KEY="your-secret-key"
$env:AWS_DEFAULT_REGION="us-east-1"
```

## How To Use

### 1. Clone The Repository

```bash
git clone https://github.com/bader2424/aws-terraform.git
cd aws-terraform
```

### 2. Go To The Dev Environment

Terraform should be run from the environment directory:

```bash
cd envs/dev
```

### 3. Review The Demo Values

Before applying, review these files:

- `modules/ec2/main.tf`
- `modules/rds/main.tf`
- `modules/iam/main.tf`
- `envs/dev/provider.tf`

Important demo values to change before a real deployment:

- Replace the placeholder EC2 AMI `ami-12345678` with a valid AMI ID for `us-east-1`.
- Replace the hard-coded RDS password.
- Restrict SSH ingress instead of allowing `0.0.0.0/0`.
- Avoid attaching `AdministratorAccess` unless it is truly required.
- Remove the provider credential-skip settings if you want normal AWS credential validation.

### 4. Initialize Terraform

```bash
terraform init
```

### 5. Format The Code

```bash
terraform fmt -recursive ../..
```

### 6. Validate The Configuration

```bash
terraform validate
```

### 7. Preview The Plan

```bash
terraform plan -out=tfplan
```

### 8. Apply The Infrastructure

```bash
terraform apply tfplan
```

If you do not save a plan file, you can also run:

```bash
terraform apply
```

### 9. View Outputs

```bash
terraform output
```

The main output is `architecture_summary`, which includes the VPC ID, subnet IDs, EC2 instance ID, ALB DNS name, S3 bucket name, RDS instance ID, and IAM user name.

### 10. Destroy The Infrastructure

To avoid ongoing AWS charges, destroy the environment when you are done:

```bash
terraform destroy
```

## Module Details

### VPC

Creates the base network:

- VPC CIDR: `10.0.0.0/16`
- Public subnets: `10.0.1.0/24`, `10.0.2.0/24`
- Private subnets: `10.0.3.0/24`, `10.0.4.0/24`
- Internet gateway.
- Public route table with a default route to the internet gateway.

### EC2

Creates:

- One EC2 instance.
- One security group.

Default instance type: `t3.micro`.

Note: the current AMI is a placeholder and must be replaced before a real apply.

### ALB

Creates:

- Public Application Load Balancer.
- Target group on port `80`.
- Target attachment for the EC2 instance.
- HTTP listener on port `80`.

### RDS

Creates:

- DB subnet group using the private subnets.
- PostgreSQL RDS instance.

Note: the current database password is hard-coded and should be replaced with a sensitive Terraform variable, AWS Secrets Manager, or another secure secret management approach.

### S3

Creates:

- S3 bucket named from the environment value.
- Bucket versioning.

Note: S3 bucket names must be globally unique. The default name `dev-bucket-demo` may already be taken in AWS.

### IAM

Creates:

- IAM user.
- IAM group.
- Group membership.
- AWS managed `AdministratorAccess` policy attachment.

Note: broad administrator access is not recommended for production. Use least-privilege policies instead.

## Configuration

Most defaults are defined inside each module. The `dev` environment passes `environment = "dev"` to every module.

Common values you may want to customize:

- AWS region in `envs/dev/provider.tf`.
- VPC CIDR and subnet CIDRs in `modules/vpc/variables.tf`.
- EC2 instance type and AMI in `modules/ec2`.
- RDS username, password, storage, and instance class in `modules/rds/main.tf`.
- S3 bucket name in `modules/s3/main.tf`.
- IAM permissions in `modules/iam/main.tf`.

## Security Notes

This repository currently contains demo-friendly settings that should be hardened before production use:

- `modules/rds/main.tf` includes a hard-coded database password.
- `modules/ec2/main.tf` allows SSH from anywhere.
- `modules/iam/main.tf` attaches `AdministratorAccess`.
- `modules/ec2/main.tf` uses a placeholder AMI.
- `envs/dev/provider.tf` skips provider credential and account validation.
- There is no remote backend configured for shared Terraform state.
- The EC2 instance is placed in a private subnet, but the ALB target group expects to reach it on port `80`; make sure the instance security group allows ALB traffic before using this as an application deployment.

## Files To Review Or Remove

The main deployable Terraform root is `envs/dev`. Because of that, these top-level files may be unnecessary unless you plan to make the repository root a separate Terraform root module:

- `provider.tf`
- `variables.tf`
- `.terraform.lock.hcl`

The useful lock file for the current workflow is:

- `envs/dev/.terraform.lock.hcl`

Keep `.gitignore`; it correctly excludes Terraform state, plan files, crash logs, and local editor folders.

## Recommended Improvements

- Move hard-coded values into variables with descriptions.
- Mark secrets as `sensitive = true` and do not commit real secret values.
- Use a remote backend such as S3 with DynamoDB locking.
- Add security groups that explicitly allow ALB-to-EC2 and EC2-to-RDS traffic.
- Add availability zones to subnet definitions for clearer multi-AZ behavior.
- Add environment-specific `terraform.tfvars` files.
- Replace broad IAM permissions with least-privilege policies.
- Add CI checks for `terraform fmt`, `terraform validate`, and security scanning.

## License

No license file is currently included. Add one if you want others to use, modify, or distribute this project.
