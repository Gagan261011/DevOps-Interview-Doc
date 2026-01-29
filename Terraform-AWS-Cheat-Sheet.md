# Terraform with AWS – High Retrieval Cheat Sheet

## Table of Contents

1. [Overview Table](#overview-table)
2. [Terraform Workflow](#terraform-workflow)
3. [Providers](#providers)
4. [State Management](#state-management)
5. [Remote Backend (S3)](#remote-backend-s3)
6. [State Locking (DynamoDB)](#state-locking-dynamodb)
7. [Resources](#resources)
8. [Data Sources](#data-sources)
9. [Variables](#variables)
10. [Outputs](#outputs)
11. [Locals](#locals)
12. [Modules](#modules)
13. [Workspaces](#workspaces)
14. [Lifecycle Meta-Arguments](#lifecycle-meta-arguments)
15. [Dependencies](#dependencies)
16. [Best Practices](#best-practices)
17. [AWS Provider](#aws-provider)
18. [IAM](#iam)
19. [VPC](#vpc)
20. [Subnets](#subnets)
21. [Security Groups](#security-groups)
22. [EC2 Instances](#ec2-instances)
23. [AMI Data Source](#ami-data-source)
24. [Key Pairs](#key-pairs)
25. [Application Load Balancer](#application-load-balancer)
26. [Auto Scaling Group](#auto-scaling-group)
27. [S3 Buckets](#s3-buckets)
28. [RDS](#rds)
29. [Tags](#tags)

---

## Overview Table

| Topic | Trigger (1 line) | Key Construct |
|-------|------------------|---------------|
| Terraform Workflow | Init, plan, apply, destroy lifecycle | `terraform init`, `terraform plan`, `terraform apply` |
| Providers | Connect Terraform to cloud platforms like AWS | `provider "aws"`, `required_providers` |
| State Management | Track infrastructure in terraform.tfstate file | `terraform.tfstate`, remote backend |
| Remote Backend (S3) | Store state file remotely in S3 bucket | `backend "s3"`, bucket, key, region |
| State Locking (DynamoDB) | Prevent concurrent state modifications | DynamoDB table, `LockID` primary key |
| Resources | Actual infrastructure components to create | `resource "type" "name"` |
| Data Sources | Query existing infrastructure without creating | `data "type" "name"` |
| Variables | Parameterize configurations for reusability | `variable "name"`, `var.name` |
| Outputs | Export values from modules or root | `output "name"`, `value` |
| Locals | Computed values within module scope | `locals {}`, `local.name` |
| Modules | Reusable Terraform code packages | `module "name"`, `source` |
| Workspaces | Manage multiple environments with same code | `terraform workspace`, default/dev/prod |
| Lifecycle Meta-Arguments | Control resource creation and destruction behavior | `create_before_destroy`, `prevent_destroy`, `ignore_changes` |
| Dependencies | Control resource creation order | `depends_on`, implicit dependencies |
| Best Practices | Industry standards for maintainable Terraform | DRY, versioning, state management |
| AWS Provider | AWS-specific Terraform provider configuration | `provider "aws"`, region, credentials |
| IAM | Identity and access management for AWS resources | `aws_iam_role`, `aws_iam_policy`, `aws_iam_instance_profile` |
| VPC | Virtual private cloud networking | `aws_vpc`, CIDR block |
| Subnets | Network segments within VPC | `aws_subnet`, public/private |
| Security Groups | Virtual firewall rules for instances | `aws_security_group`, ingress/egress |
| EC2 Instances | Virtual machines in AWS cloud | `aws_instance`, AMI, instance_type |
| AMI Data Source | Fetch existing Amazon Machine Image IDs | `data "aws_ami"`, filter |
| Key Pairs | SSH key pairs for EC2 access | `aws_key_pair`, public_key |
| Application Load Balancer | Distribute traffic across multiple targets | `aws_lb`, `aws_lb_target_group`, `aws_lb_listener` |
| Auto Scaling Group | Automatically scale EC2 instances | `aws_autoscaling_group`, min/max/desired |
| S3 Buckets | Object storage service buckets | `aws_s3_bucket`, bucket name |
| RDS | Managed relational database service | `aws_db_instance`, engine, instance_class |
| Tags | Metadata labels for AWS resources | `tags = { Name = "value" }` |

---

## Terraform Workflow

**Primary Trigger:** How do you deploy infrastructure with Terraform from scratch to production?

**What it is**
- Standard sequence of commands to provision infrastructure
- Four main stages: initialize, preview, apply, destroy
- Ensures predictable and repeatable deployments

**Why / When used**
- Every Terraform project starts with init to download providers
- Use plan before apply to review changes and catch errors
- Apply creates or updates real infrastructure in cloud
- Destroy removes all managed resources when decommissioning

**Key constructs**
- `terraform init`
- `terraform plan`
- `terraform apply`
- `terraform destroy`
- `terraform validate`
- `terraform fmt`

**Minimal Terraform example**

```hcl
# main.tf
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = {
    Name = "WebServer"
  }
}
```

**Commands to run:**
```bash
terraform init      # Download AWS provider
terraform validate  # Check syntax
terraform plan      # Preview changes
terraform apply     # Create EC2 instance
terraform destroy   # Remove EC2 instance
```

**Example scenario**
- You write Terraform config for an EC2 instance
- Run init to set up the working directory and download providers
- Run plan to see what will be created before spending money
- Run apply to provision the actual infrastructure

**KT / interview line**
"Terraform workflow follows init-plan-apply-destroy cycle where init downloads providers, plan shows what changes will happen, apply provisions infrastructure, and destroy removes everything."

**Closing trigger**
Always run plan before apply to avoid surprises in production.

---

## Providers

**Primary Trigger:** How does Terraform know to create resources in AWS versus Azure or GCP?

**What it is**
- Plugins that enable Terraform to interact with cloud platforms
- Each provider offers resource types and data sources
- Must be declared and configured before use

**Why / When used**
- Every Terraform project needs at least one provider
- AWS provider requires region and optionally credentials
- Multiple providers can be used in single configuration
- Provider versions should be pinned for consistency

**Key constructs**
- `provider "aws"`
- `required_providers`
- `version` constraints
- `alias` for multiple instances

**Minimal Terraform example**

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
  region = "us-east-1"
}

# Use multiple regions
provider "aws" {
  alias  = "west"
  region = "us-west-2"
}

resource "aws_instance" "east" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}

resource "aws_instance" "west" {
  provider      = aws.west
  ami           = "ami-0d1cd67c26f5fca19"
  instance_type = "t2.micro"
}
```

**Example scenario**
- You need to deploy resources in multiple AWS regions
- Configure primary provider for us-east-1
- Add aliased provider for us-west-2
- Reference the alias when creating region-specific resources

**KT / interview line**
"Providers are plugins that let Terraform communicate with APIs like AWS, and you configure them with credentials and settings like region before creating any resources."

**Closing trigger**
Always pin provider versions in production to avoid breaking changes.

---

## State Management

**Primary Trigger:** How does Terraform know what infrastructure already exists in AWS?

**What it is**
- JSON file tracking all managed infrastructure
- Maps Terraform config to real-world resources
- Stores resource attributes and metadata

**Why / When used**
- Terraform compares desired state (config) to current state (tfstate)
- Used to calculate what changes need to be made
- Critical for team collaboration and preventing drift
- State file contains sensitive data and must be protected

**Key constructs**
- `terraform.tfstate`
- `terraform.tfstate.backup`
- `terraform state list`
- `terraform state show`
- `terraform refresh`

**Minimal Terraform example**

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_s3_bucket" "data" {
  bucket = "my-app-data-bucket-12345"
  
  tags = {
    Environment = "production"
  }
}

# After apply, state file tracks:
# - Bucket ARN
# - Creation date
# - Region
# - All attributes
```

**State commands:**
```bash
terraform state list                    # List all resources
terraform state show aws_s3_bucket.data # Show specific resource
terraform state pull                    # Download remote state
terraform state rm aws_s3_bucket.data   # Remove from state
```

**Example scenario**
- You create an S3 bucket with Terraform
- State file stores bucket name, ARN, and properties
- Next time you run plan, Terraform reads state to see bucket exists
- Only applies changes if config differs from state

**KT / interview line**
"Terraform state is a JSON file that maps your configuration to real resources in AWS, allowing Terraform to know what exists and calculate what needs to change."

**Closing trigger**
Never manually edit state files, always use Terraform commands.

---

## Remote Backend (S3)

**Primary Trigger:** How do you share Terraform state across your team without committing tfstate to Git?

**What it is**
- Store state file in S3 bucket instead of local disk
- Enables team collaboration with centralized state
- Supports state locking and versioning

**Why / When used**
- Local state doesn't work for teams or CI/CD pipelines
- S3 backend provides durability and availability
- Enables state locking with DynamoDB integration
- Required for production infrastructure management

**Key constructs**
- `backend "s3"`
- `bucket` name
- `key` path in bucket
- `region`
- `dynamodb_table` for locking

**Minimal Terraform example**

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "prod/vpc/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-state-lock"
    encrypt        = true
  }
}

provider "aws" {
  region = "us-east-1"
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  
  tags = {
    Name = "Production VPC"
  }
}
```

**Pre-requisites (create manually or separate config):**
```bash
# S3 bucket must exist
aws s3api create-bucket --bucket my-terraform-state-bucket --region us-east-1

# Enable versioning
aws s3api put-bucket-versioning --bucket my-terraform-state-bucket --versioning-configuration Status=Enabled
```

**Example scenario**
- Your team of five engineers works on same infrastructure
- Configure S3 backend so everyone reads/writes same state
- State stored at s3://bucket/prod/vpc/terraform.tfstate
- Each team member runs terraform commands locally, state syncs automatically

**KT / interview line**
"S3 backend stores Terraform state remotely in an S3 bucket, enabling team collaboration and providing durability, versioning, and encryption for state files."

**Closing trigger**
Always enable S3 versioning to recover from state corruption.

---

## State Locking (DynamoDB)

**Primary Trigger:** How do you prevent two engineers from running terraform apply simultaneously?

**What it is**
- DynamoDB table that locks state during operations
- Prevents concurrent modifications to infrastructure
- Automatically managed by Terraform with S3 backend

**Why / When used**
- Without locking, two applies can corrupt state or create conflicts
- S3 backend supports locking via DynamoDB table
- Table only needs LockID attribute as primary key
- Terraform automatically acquires and releases locks

**Key constructs**
- DynamoDB table name
- `LockID` string attribute (partition key)
- `dynamodb_table` in backend config
- `terraform force-unlock` for stuck locks

**Minimal Terraform example**

```hcl
# Step 1: Create DynamoDB table for locking
resource "aws_dynamodb_table" "terraform_lock" {
  name           = "terraform-state-lock"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "LockID"
  
  attribute {
    name = "LockID"
    type = "S"
  }
  
  tags = {
    Name = "Terraform State Lock"
  }
}

# Step 2: Use in backend configuration
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-state-lock"
    encrypt        = true
  }
}
```

**Example scenario**
- Engineer A runs terraform apply for VPC changes
- Terraform creates lock entry in DynamoDB table
- Engineer B tries terraform apply, gets lock error
- After Engineer A's apply completes, lock released automatically
- Engineer B can now run apply

**KT / interview line**
"State locking uses a DynamoDB table with LockID attribute to prevent concurrent Terraform operations, ensuring only one person can modify infrastructure at a time."

**Closing trigger**
Create the DynamoDB table before configuring it in backend.

---

## Resources

**Primary Trigger:** What is the difference between declaring something in Terraform versus using AWS console?

**What it is**
- Infrastructure components that Terraform creates and manages
- Each resource has a type and local name
- Terraform creates, updates, or deletes resources based on config

**Why / When used**
- Core building blocks of infrastructure as code
- Every AWS service has corresponding resource types
- Resources can reference each other for relationships
- Changes to resource block trigger updates in AWS

**Key constructs**
- `resource "type" "name"`
- Resource arguments (configuration)
- Resource attributes (computed values)
- Resource references `type.name.attribute`

**Minimal Terraform example**

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  
  tags = {
    Name = "Main VPC"
  }
}

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id  # Reference
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true
  
  tags = {
    Name = "Public Subnet"
  }
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.public.id
  
  tags = {
    Name = "WebServer"
  }
}
```

**Example scenario**
- You declare VPC, subnet, and EC2 instance as resources
- Terraform creates VPC first, then subnet (depends on VPC ID)
- EC2 instance uses subnet ID automatically
- If you change instance_type and apply, Terraform updates the instance

**KT / interview line**
"Resources are the infrastructure components Terraform manages in AWS, declared with resource blocks that specify the type, name, and configuration arguments."

**Closing trigger**
Every AWS component you create should be a resource block.

---

## Data Sources

**Primary Trigger:** How do you reference existing AWS resources that weren't created by Terraform?

**What it is**
- Read-only queries to fetch information about existing infrastructure
- Does not create or modify anything in AWS
- Used to integrate with manually created or external resources

**Why / When used**
- Reference existing VPCs, AMIs, or subnets not managed by Terraform
- Get dynamic values like latest AMI ID for region
- Avoid hardcoding values that can be looked up
- Connect new infrastructure to existing resources

**Key constructs**
- `data "type" "name"`
- Query filters and search parameters
- Attributes accessed via `data.type.name.attribute`
- Common types: aws_ami, aws_vpc, aws_subnet, aws_availability_zones

**Minimal Terraform example**

```hcl
provider "aws" {
  region = "us-east-1"
}

# Fetch latest Amazon Linux 2 AMI
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
  
  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

# Fetch existing VPC by tag
data "aws_vpc" "existing" {
  tags = {
    Name = "Legacy-VPC"
  }
}

# Use data source values in resources
resource "aws_instance" "app" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t2.micro"
  subnet_id     = data.aws_vpc.existing.id
  
  tags = {
    Name = "AppServer"
  }
}
```

**Example scenario**
- Your company has existing VPC created manually
- You need to launch EC2 instances in that VPC
- Use data source to query VPC by name tag
- Reference VPC ID in your instance configuration without hardcoding

**KT / interview line**
"Data sources query existing infrastructure in AWS without modifying it, allowing you to reference resources like AMIs or VPCs that weren't created by your Terraform code."

**Closing trigger**
Use data sources to avoid hardcoding IDs and fetch dynamic values.

---

## Variables

**Primary Trigger:** How do you make Terraform configurations reusable across environments like dev, staging, and prod?

**What it is**
- Input parameters that customize Terraform configurations
- Declared with type, description, and optional default
- Values passed via CLI, tfvars files, or environment variables

**Why / When used**
- Eliminate hardcoded values for different environments
- Reuse same config with different values per environment
- Enforce types and validation rules
- Document expected inputs for modules

**Key constructs**
- `variable "name"`
- `type` (string, number, bool, list, map, object)
- `default` value
- `description`
- `var.name` to reference
- `terraform.tfvars` for values

**Minimal Terraform example**

```hcl
# variables.tf
variable "environment" {
  description = "Environment name"
  type        = string
  default     = "dev"
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

variable "instance_count" {
  description = "Number of instances"
  type        = number
  default     = 1
  
  validation {
    condition     = var.instance_count > 0 && var.instance_count <= 10
    error_message = "Instance count must be between 1 and 10."
  }
}

variable "tags" {
  description = "Common tags"
  type        = map(string)
  default = {
    Managed = "Terraform"
  }
}

# main.tf
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "app" {
  count         = var.instance_count
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = var.instance_type
  
  tags = merge(var.tags, {
    Name        = "App-${var.environment}-${count.index + 1}"
    Environment = var.environment
  })
}
```

**Usage:**
```bash
# Use defaults
terraform apply

# Override via CLI
terraform apply -var="environment=prod" -var="instance_count=3"

# Use tfvars file
# prod.tfvars
environment    = "prod"
instance_type  = "t3.medium"
instance_count = 5

terraform apply -var-file="prod.tfvars"
```

**Example scenario**
- You manage dev, staging, and prod environments
- Same infrastructure but different instance sizes
- Create variables for environment, instance_type, instance_count
- Use prod.tfvars with large instances and dev.tfvars with small instances

**KT / interview line**
"Variables parameterize Terraform configurations for reusability, allowing the same code to deploy different environments by passing different values via CLI, tfvars files, or environment variables."

**Closing trigger**
Always add descriptions and types to variables for clarity.

---

## Outputs

**Primary Trigger:** How do you get information like EC2 public IP or RDS endpoint after Terraform creates resources?

**What it is**
- Values exported from Terraform modules or root configuration
- Displayed after apply completes
- Can be queried with terraform output command

**Why / When used**
- Show important information like IPs, endpoints, ARNs
- Pass values between modules
- Feed outputs to other tools or scripts
- Document what was created for operations team

**Key constructs**
- `output "name"`
- `value` expression
- `description`
- `sensitive` flag to hide values
- `terraform output` command

**Minimal Terraform example**

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = {
    Name = "WebServer"
  }
}

resource "aws_eip" "web_ip" {
  instance = aws_instance.web.id
  domain   = "vpc"
}

output "instance_id" {
  description = "EC2 instance ID"
  value       = aws_instance.web.id
}

output "instance_public_ip" {
  description = "Public IP address"
  value       = aws_eip.web_ip.public_ip
}

output "instance_private_ip" {
  description = "Private IP address"
  value       = aws_instance.web.private_ip
}

output "ssh_command" {
  description = "SSH command to connect"
  value       = "ssh -i my-key.pem ec2-user@${aws_eip.web_ip.public_ip}"
}
```

**After apply:**
```bash
Outputs:

instance_id = "i-0123456789abcdef0"
instance_public_ip = "54.123.45.67"
instance_private_ip = "10.0.1.25"
ssh_command = "ssh -i my-key.pem ec2-user@54.123.45.67"

# Query specific output
terraform output instance_public_ip
# 54.123.45.67

# JSON format
terraform output -json
```

**Example scenario**
- You deploy web server with Terraform
- Need to give public IP to team for testing
- Add output for public IP address
- After apply, IP displayed automatically without checking AWS console

**KT / interview line**
"Outputs export values from Terraform configurations, displaying important information like IPs or endpoints after apply and enabling data flow between modules."

**Closing trigger**
Use outputs to document and share important resource attributes.

---

## Locals

**Primary Trigger:** How do you avoid repeating the same expression multiple times in Terraform?

**What it is**
- Named values computed within a module
- Similar to local variables in programming languages
- Evaluated once and reused throughout configuration

**Why / When used**
- Avoid repeating complex expressions
- Create derived values from variables
- Improve readability with descriptive names
- Compute common tags or naming conventions

**Key constructs**
- `locals {}` block
- `local.name` to reference
- Can reference variables, resources, data sources
- Cannot be overridden from outside

**Minimal Terraform example**

```hcl
variable "environment" {
  type    = string
  default = "dev"
}

variable "project" {
  type    = string
  default = "webapp"
}

locals {
  # Common tags used across all resources
  common_tags = {
    Environment = var.environment
    Project     = var.project
    ManagedBy   = "Terraform"
    Owner       = "DevOps Team"
  }
  
  # Naming convention
  name_prefix = "${var.project}-${var.environment}"
  
  # Computed values
  is_production = var.environment == "prod"
  instance_type = local.is_production ? "t3.medium" : "t2.micro"
}

provider "aws" {
  region = "us-east-1"
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-vpc"
  })
}

resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-public-subnet"
  })
}

resource "aws_instance" "app" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = local.instance_type
  subnet_id     = aws_subnet.public.id
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-app-server"
  })
}
```

**Example scenario**
- All resources need same set of tags for billing and organization
- Different instance types for prod vs non-prod
- Define common_tags once in locals block
- Merge common_tags into every resource without repeating the map

**KT / interview line**
"Locals define named computed values within a module to avoid repeating expressions, improving code readability and maintainability by centralizing common values like tags or naming patterns."

**Closing trigger**
Use locals for values used multiple times, not for single-use expressions.

---

## Modules

**Primary Trigger:** How do you package and reuse Terraform configurations across multiple projects?

**What it is**
- Reusable packages of Terraform configuration files
- Encapsulate resources and logic into single unit
- Can be sourced from local paths, Git, or Terraform Registry

**Why / When used**
- Avoid duplicating code across projects
- Standardize infrastructure patterns across teams
- Create company-specific infrastructure templates
- Use community modules for common patterns like VPC or EKS

**Key constructs**
- `module "name"` block
- `source` path (local, Git, Registry)
- Input variables to module
- Output values from module
- `terraform init` downloads modules

**Minimal Terraform example**

```hcl
# modules/ec2-instance/variables.tf
variable "instance_name" {
  type = string
}

variable "instance_type" {
  type    = string
  default = "t2.micro"
}

variable "subnet_id" {
  type = string
}

# modules/ec2-instance/main.tf
resource "aws_instance" "this" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = var.instance_type
  subnet_id     = var.subnet_id
  
  tags = {
    Name = var.instance_name
  }
}

# modules/ec2-instance/outputs.tf
output "instance_id" {
  value = aws_instance.this.id
}

output "private_ip" {
  value = aws_instance.this.private_ip
}

# Root main.tf
provider "aws" {
  region = "us-east-1"
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"
}

# Use module multiple times
module "web_server" {
  source = "./modules/ec2-instance"
  
  instance_name = "WebServer"
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.public.id
}

module "app_server" {
  source = "./modules/ec2-instance"
  
  instance_name = "AppServer"
  instance_type = "t2.small"
  subnet_id     = aws_subnet.public.id
}

# Access module outputs
output "web_instance_id" {
  value = module.web_server.instance_id
}

output "app_private_ip" {
  value = module.app_server.private_ip
}
```

**Using registry module:**
```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.1.2"
  
  name = "my-vpc"
  cidr = "10.0.0.0/16"
  
  azs             = ["us-east-1a", "us-east-1b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]
  
  enable_nat_gateway = true
}
```

**Example scenario**
- Your company deploys same EC2 pattern in every project
- Create module with standard security settings and tags
- Each project calls module with different parameters
- Updates to module automatically benefit all projects

**KT / interview line**
"Modules are reusable Terraform configuration packages that encapsulate resources and logic, enabling code reuse across projects and standardization of infrastructure patterns."

**Closing trigger**
Create modules when you find yourself copying the same code twice.

---

## Workspaces

**Primary Trigger:** How do you manage multiple environments using the same Terraform code?

**What it is**
- Named state containers within same configuration
- Each workspace has separate state file
- Default workspace created automatically

**Why / When used**
- Manage dev, staging, prod with single codebase
- Test changes in isolated state before production
- Quick environment switching without changing backend config
- Alternative to separate directories per environment

**Key constructs**
- `terraform workspace` commands
- `terraform.workspace` variable
- `default` workspace
- State files stored per workspace

**Minimal Terraform example**

```hcl
provider "aws" {
  region = "us-east-1"
}

locals {
  environment = terraform.workspace
  
  # Different configs per workspace
  instance_config = {
    dev = {
      instance_type = "t2.micro"
      count         = 1
    }
    staging = {
      instance_type = "t2.small"
      count         = 2
    }
    prod = {
      instance_type = "t3.medium"
      count         = 5
    }
  }
  
  config = local.instance_config[local.environment]
}

resource "aws_instance" "app" {
  count         = local.config.count
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = local.config.instance_type
  
  tags = {
    Name        = "App-${local.environment}-${count.index + 1}"
    Environment = local.environment
    Workspace   = terraform.workspace
  }
}

output "environment" {
  value = "Currently in ${terraform.workspace} workspace"
}
```

**Workspace commands:**
```bash
# List workspaces
terraform workspace list
# * default

# Create and switch to dev
terraform workspace new dev
terraform apply  # Creates 1 t2.micro

# Create and switch to prod
terraform workspace new prod
terraform apply  # Creates 5 t3.medium

# Switch between workspaces
terraform workspace select dev

# Show current workspace
terraform workspace show
# dev

# Delete workspace (must be empty)
terraform workspace delete staging
```

**Example scenario**
- You maintain dev and prod environments
- Same VPC structure but different sizes
- Create dev workspace, apply small infrastructure
- Create prod workspace, apply large infrastructure
- Each workspace maintains separate state

**KT / interview line**
"Workspaces allow managing multiple environments with the same Terraform code by maintaining separate state files, useful for dev/staging/prod separation without duplicating configurations."

**Closing trigger**
Use workspaces for simple multi-environment setups, directories for complex ones.

---

## Lifecycle Meta-Arguments

**Primary Trigger:** How do you prevent Terraform from destroying and recreating a database during updates?

**What it is**
- Special configuration options controlling resource behavior
- Modify default create-update-destroy behavior
- Apply to any resource type

**Why / When used**
- Prevent data loss during replacement operations
- Protect critical resources from accidental deletion
- Ignore certain attribute changes from drift
- Control resource creation order during replacements

**Key constructs**
- `create_before_destroy`
- `prevent_destroy`
- `ignore_changes`
- `replace_triggered_by`

**Minimal Terraform example**

```hcl
provider "aws" {
  region = "us-east-1"
}

# Prevent accidental deletion of production database
resource "aws_db_instance" "production" {
  identifier           = "prod-db"
  engine               = "mysql"
  instance_class       = "db.t3.micro"
  allocated_storage    = 20
  username             = "admin"
  password             = "changeme123"  # Use secrets manager in real scenario
  skip_final_snapshot  = true
  
  lifecycle {
    prevent_destroy = true  # Error if trying to destroy
  }
}

# Create new instance before destroying old one
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = {
    Name = "WebServer"
  }
  
  lifecycle {
    create_before_destroy = true  # Zero downtime replacement
  }
}

# Ignore tag changes made outside Terraform
resource "aws_instance" "app" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = {
    Name        = "AppServer"
    Environment = "prod"
    # Other teams add tags via console
  }
  
  lifecycle {
    ignore_changes = [
      tags["LastModified"],  # Ignore specific tag
      tags,                  # Or ignore all tags
    ]
  }
}

# S3 bucket with multiple lifecycle rules
resource "aws_s3_bucket" "data" {
  bucket = "my-data-bucket-12345"
  
  lifecycle {
    prevent_destroy = true
    ignore_changes = [
      tags,
      lifecycle_rule,  # Allow manual lifecycle rule changes
    ]
  }
}
```

**Example scenario**
- Your RDS database contains production data
- Adding prevent_destroy ensures terraform destroy fails
- When changing instance type, use create_before_destroy
- New instance created first, traffic switches, old one deleted
- Zero downtime for users during infrastructure updates

**KT / interview line**
"Lifecycle meta-arguments control resource behavior, with create_before_destroy for zero-downtime replacements, prevent_destroy to protect critical resources, and ignore_changes to prevent drift detection."

**Closing trigger**
Always use prevent_destroy on stateful resources like databases.

---

## Dependencies

**Primary Trigger:** How do you ensure VPC is created before subnets when both are in same Terraform config?

**What it is**
- Control the order in which Terraform creates resources
- Implicit dependencies from resource references
- Explicit dependencies using depends_on

**Why / When used**
- Terraform automatically detects most dependencies
- Use depends_on for non-obvious dependencies
- Ensure IAM policies exist before EC2 uses them
- Wait for VPC endpoints before launching private instances

**Key constructs**
- `depends_on` meta-argument
- Implicit dependencies via references
- Resource graph
- `terraform graph` command

**Minimal Terraform example**

```hcl
provider "aws" {
  region = "us-east-1"
}

# Implicit dependency: subnet depends on VPC
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  
  tags = {
    Name = "Main VPC"
  }
}

resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id  # Implicit dependency
  cidr_block = "10.0.1.0/24"
  
  tags = {
    Name = "Public Subnet"
  }
}

# IAM resources
resource "aws_iam_role" "ec2_role" {
  name = "ec2-role"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "ec2.amazonaws.com"
      }
    }]
  })
}

resource "aws_iam_role_policy_attachment" "s3_access" {
  role       = aws_iam_role.ec2_role.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"
}

resource "aws_iam_instance_profile" "ec2_profile" {
  name = "ec2-profile"
  role = aws_iam_role.ec2_role.name
}

# Explicit dependency example
resource "aws_instance" "app" {
  ami                  = "ami-0c55b159cbfafe1f0"
  instance_type        = "t2.micro"
  subnet_id            = aws_subnet.public.id
  iam_instance_profile = aws_iam_instance_profile.ec2_profile.name
  
  # Ensure policy attached before instance starts
  depends_on = [
    aws_iam_role_policy_attachment.s3_access
  ]
  
  tags = {
    Name = "AppServer"
  }
}

# S3 bucket that must exist before instance
resource "aws_s3_bucket" "app_data" {
  bucket = "app-data-bucket-12345"
}

resource "aws_instance" "worker" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.public.id
  
  # Worker needs bucket to exist on boot
  depends_on = [
    aws_s3_bucket.app_data
  ]
  
  user_data = <<-EOF
              #!/bin/bash
              aws s3 ls s3://${aws_s3_bucket.app_data.bucket}
              EOF
  
  tags = {
    Name = "Worker"
  }
}
```

**View dependency graph:**
```bash
terraform graph | dot -Tpng > graph.png
```

**Example scenario**
- EC2 instance needs IAM role and S3 bucket ready at boot
- Subnet ID reference creates implicit dependency on VPC
- IAM policy attachment has no direct reference in instance
- Use depends_on to ensure policy attached before instance launches

**KT / interview line**
"Terraform automatically detects implicit dependencies when resources reference each other, but depends_on is used for explicit dependencies where the relationship isn't obvious from references."

**Closing trigger**
Let Terraform handle dependencies automatically, use depends_on only when necessary.

---

## Best Practices

**Primary Trigger:** What are the key principles for writing production-grade Terraform code?

**What it is**
- Industry standards for maintainable infrastructure code
- Patterns that prevent common mistakes
- Guidelines for team collaboration

**Why / When used**
- Production infrastructure requires reliability and safety
- Teams need consistent code style and structure
- Avoid state corruption and drift
- Enable code review and collaboration

**Key constructs**
- Remote state with locking
- Version pinning
- Module structure
- Code organization
- Security practices

**Minimal Terraform example**

```hcl
# 1. Pin Terraform and provider versions
terraform {
  required_version = ">= 1.5.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  
  # 2. Use remote backend with locking
  backend "s3" {
    bucket         = "company-terraform-state"
    key            = "prod/vpc/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}

# 3. Use variables for everything that changes
variable "environment" {
  description = "Environment name"
  type        = string
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "vpc_cidr" {
  description = "VPC CIDR block"
  type        = string
  default     = "10.0.0.0/16"
}

# 4. Use locals for common values
locals {
  common_tags = {
    Environment = var.environment
    ManagedBy   = "Terraform"
    Project     = "webapp"
  }
  
  name_prefix = "webapp-${var.environment}"
}

# 5. Organize resources logically
provider "aws" {
  region = "us-east-1"
  
  default_tags {
    tags = local.common_tags
  }
}

# 6. Use data sources for existing resources
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

# 7. Create resources with clear names
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  
  tags = {
    Name = "${local.name_prefix}-vpc"
  }
}

# 8. Use lifecycle rules for critical resources
resource "aws_db_instance" "main" {
  identifier        = "${local.name_prefix}-db"
  engine            = "postgres"
  instance_class    = "db.t3.micro"
  allocated_storage = 20
  
  lifecycle {
    prevent_destroy = true
  }
}

# 9. Output important values
output "vpc_id" {
  description = "VPC ID for reference"
  value       = aws_vpc.main.id
}

# 10. Never commit sensitive data
# Use AWS Secrets Manager or SSM Parameter Store
data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "prod/db/password"
}
```

**File structure best practices:**
```
project/
├── main.tf           # Primary resources
├── variables.tf      # Input variables
├── outputs.tf        # Output values
├── versions.tf       # Provider versions
├── backend.tf        # Backend configuration
├── data.tf           # Data sources
├── locals.tf         # Local values
├── terraform.tfvars  # Default values (not committed)
├── prod.tfvars       # Production values
└── modules/          # Reusable modules
    └── vpc/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

**Example scenario**
- Team of engineers working on production infrastructure
- Remote state prevents conflicts
- Version pinning prevents surprise provider updates
- Common tags automatically applied to all resources
- Separate tfvars files for each environment
- Modules ensure consistent patterns across projects

**KT / interview line**
"Terraform best practices include remote state with locking, version pinning, using variables for flexibility, organizing code into modules, and never committing sensitive data or state files."

**Closing trigger**
Follow these practices from day one to avoid painful refactoring later.

---

## AWS Provider

**Primary Trigger:** How do you configure Terraform to authenticate and interact with AWS?

**What it is**
- Official HashiCorp plugin for AWS infrastructure
- Handles authentication and API communication
- Required for all AWS resource management

**Why / When used**
- First step in any AWS Terraform project
- Configure region where resources will be created
- Set up authentication via credentials or IAM roles
- Can configure multiple providers for multi-region deployments

**Key constructs**
- `provider "aws"`
- `region` attribute
- `profile` for AWS CLI profiles
- `assume_role` for cross-account access
- `default_tags` for common tags

**Minimal Terraform example**

```hcl
# Basic provider configuration
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

# Using AWS CLI profile
provider "aws" {
  region  = "us-east-1"
  profile = "production"
}

# Multi-region setup
provider "aws" {
  region = "us-east-1"
  alias  = "primary"
}

provider "aws" {
  region = "us-west-2"
  alias  = "disaster_recovery"
}

# Cross-account access
provider "aws" {
  region = "us-east-1"
  
  assume_role {
    role_arn = "arn:aws:iam::123456789012:role/TerraformRole"
  }
}

# Default tags for all resources
provider "aws" {
  region = "us-east-1"
  
  default_tags {
    tags = {
      ManagedBy   = "Terraform"
      Environment = "production"
      CostCenter  = "Engineering"
    }
  }
}

# Using provider aliases
resource "aws_instance" "east" {
  provider      = aws.primary
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}

resource "aws_instance" "west" {
  provider      = aws.disaster_recovery
  ami           = "ami-0d1cd67c26f5fca19"
  instance_type = "t2.micro"
}
```

**Authentication methods (priority order):**
```bash
# 1. Environment variables
export AWS_ACCESS_KEY_ID="AKIAIOSFODNN7EXAMPLE"
export AWS_SECRET_ACCESS_KEY="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
export AWS_DEFAULT_REGION="us-east-1"

# 2. Shared credentials file (~/.aws/credentials)
[default]
aws_access_key_id = AKIAIOSFODNN7EXAMPLE
aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

# 3. IAM role (when running on EC2, ECS, Lambda)
# Automatic, no credentials needed

# 4. Provider configuration (not recommended)
provider "aws" {
  access_key = "AKIAIOSFODNN7EXAMPLE"  # Don't do this
  secret_key = "wJalrXUtnFEMI/..."     # Use environment variables instead
}
```

**Example scenario**
- You need to deploy infrastructure in us-east-1
- Configure AWS provider with region
- Terraform uses AWS credentials from CLI or environment
- All resources created will be in specified region
- Use alias for multi-region deployments like DR setup

**KT / interview line**
"AWS provider configures Terraform to authenticate and communicate with AWS APIs, requiring region specification and credentials from environment variables, CLI profiles, or IAM roles."

**Closing trigger**
Always pin the AWS provider version to avoid breaking changes.

---

## IAM

**Primary Trigger:** How do you create IAM roles and policies for EC2 instances to access S3?

**What it is**
- Identity and Access Management for AWS resources
- Define who can do what with which AWS resources
- Roles, policies, users, and groups

**Why / When used**
- Grant EC2 instances permissions to access AWS services
- Create least-privilege access policies
- Enable cross-account access
- Attach roles to EC2, Lambda, ECS tasks

**Key constructs**
- `aws_iam_role`
- `aws_iam_policy`
- `aws_iam_role_policy_attachment`
- `aws_iam_instance_profile`
- JSON policy documents

**Minimal Terraform example**

```hcl
provider "aws" {
  region = "us-east-1"
}

# IAM role for EC2 instances
resource "aws_iam_role" "ec2_s3_access" {
  name = "ec2-s3-access-role"
  
  # Trust policy: who can assume this role
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "ec2.amazonaws.com"
      }
    }]
  })
  
  tags = {
    Name = "EC2 S3 Access Role"
  }
}

# Custom policy for S3 access
resource "aws_iam_policy" "s3_read_write" {
  name        = "s3-read-write-policy"
  description = "Allow read/write to specific S3 bucket"
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Action = [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ]
      Resource = [
        "arn:aws:s3:::my-app-bucket",
        "arn:aws:s3:::my-app-bucket/*"
      ]
    }]
  })
}

# Attach policy to role
resource "aws_iam_role_policy_attachment" "ec2_s3_attach" {
  role       = aws_iam_role.ec2_s3_access.name
  policy_arn = aws_iam_policy.s3_read_write.arn
}

# Or use AWS managed policy
resource "aws_iam_role_policy_attachment" "cloudwatch_attach" {
  role       = aws_iam_role.ec2_s3_access.name
  policy_arn = "arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy"
}

# Instance profile (wrapper for role)
resource "aws_iam_instance_profile" "ec2_profile" {
  name = "ec2-s3-profile"
  role = aws_iam_role.ec2_s3_access.name
}

# EC2 instance using the role
resource "aws_instance" "app" {
  ami                  = "ami-0c55b159cbfafe1f0"
  instance_type        = "t2.micro"
  iam_instance_profile = aws_iam_instance_profile.ec2_profile.name
  
  tags = {
    Name = "AppServer"
  }
}

# S3 bucket the instance can access
resource "aws_s3_bucket" "app_data" {
  bucket = "my-app-bucket"
}
```

**Example scenario**
- Your application on EC2 needs to read/write to S3
- Create IAM role with S3 permissions policy
- Attach role to EC2 via instance profile
- Application uses AWS SDK without hardcoded credentials
- Instance automatically gets temporary credentials from IAM

**KT / interview line**
"IAM in Terraform creates roles, policies, and instance profiles to grant AWS resources permissions, using assume role policies to define trust and permission policies to define allowed actions."

**Closing trigger**
Always use IAM roles for EC2 instead of hardcoding credentials.

---

## VPC

**Primary Trigger:** How do you create an isolated network in AWS for your infrastructure?

**What it is**
- Virtual Private Cloud, isolated network in AWS
- Define IP address range with CIDR block
- Foundation for all networking in AWS

**Why / When used**
- Required for EC2, RDS, and most AWS resources
- Isolate production from development networks
- Control network routing and security
- Connect to on-premises networks via VPN or Direct Connect

**Key constructs**
- `aws_vpc`
- `cidr_block` (IP address range)
- `enable_dns_hostnames`
- `enable_dns_support`
- `tags` for identification

**Minimal Terraform example**

```hcl
provider "aws" {
  region = "us-east-1"
}

# Basic VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name = "Main VPC"
  }
}

# Internet Gateway for public internet access
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = {
    Name = "Main IGW"
  }
}

# Route table for public subnets
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
  
  tags = {
    Name = "Public Route Table"
  }
}

# Output VPC details
output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}

output "vpc_cidr" {
  description = "VPC CIDR block"
  value       = aws_vpc.main.cidr_block
}
```

**Common CIDR blocks:**
```
10.0.0.0/16    - 65,536 IPs (recommended for large environments)
10.0.0.0/20    - 4,096 IPs (medium environments)
10.0.0.0/24    - 256 IPs (small environments)
172.16.0.0/16  - Alternative private range
192.168.0.0/16 - Home network range
```

**Example scenario**
- You're building new application infrastructure
- Create VPC with 10.0.0.0/16 giving 65K addresses
- Enable DNS hostnames so instances get DNS names
- Attach Internet Gateway for instances to reach internet
- Create route table directing internet traffic to gateway

**KT / interview line**
"VPC creates an isolated virtual network in AWS with defined IP address range, enabling you to launch resources in a logically isolated section with control over network configuration."

**Closing trigger**
Always enable DNS hostnames for easier instance identification.

---

## Subnets

**Primary Trigger:** How do you divide a VPC into smaller network segments for different purposes?

**What it is**
- Subdivisions of VPC IP address space
- Each subnet resides in single availability zone
- Can be public (internet-accessible) or private

**Why / When used**
- Separate public-facing from internal resources
- Deploy across multiple AZs for high availability
- Control routing and access at subnet level
- Meet compliance requirements for network segregation

**Key constructs**
- `aws_subnet`
- `vpc_id` reference
- `cidr_block` (subset of VPC CIDR)
- `availability_zone`
- `map_public_ip_on_launch` for public subnets

**Minimal Terraform example**

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  
  tags = {
    Name = "Main VPC"
  }
}

# Public subnet in AZ 1
resource "aws_subnet" "public_1" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true
  
  tags = {
    Name = "Public Subnet AZ1"
    Type = "Public"
  }
}

# Public subnet in AZ 2 (for high availability)
resource "aws_subnet" "public_2" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.2.0/24"
  availability_zone       = "us-east-1b"
  map_public_ip_on_launch = true
  
  tags = {
    Name = "Public Subnet AZ2"
    Type = "Public"
  }
}

# Private subnet in AZ 1
resource "aws_subnet" "private_1" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.11.0/24"
  availability_zone = "us-east-1a"
  
  tags = {
    Name = "Private Subnet AZ1"
    Type = "Private"
  }
}

# Private subnet in AZ 2
resource "aws_subnet" "private_2" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.12.0/24"
  availability_zone = "us-east-1b"
  
  tags = {
    Name = "Private Subnet AZ2"
    Type = "Private"
  }
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = {
    Name = "Main IGW"
  }
}

# Route table for public subnets
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
  
  tags = {
    Name = "Public Route Table"
  }
}

# Associate public subnets with public route table
resource "aws_route_table_association" "public_1" {
  subnet_id      = aws_subnet.public_1.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "public_2" {
  subnet_id      = aws_subnet.public_2.id
  route_table_id = aws_route_table.public.id
}
```

**Example scenario**
- You need web servers accessible from internet and databases isolated
- Create two public subnets in different AZs for web tier
- Create two private subnets in same AZs for database tier
- Public subnets get IGW route, private subnets don't
- High availability achieved by spreading across multiple AZs

**KT / interview line**
"Subnets divide VPC CIDR into smaller ranges within specific availability zones, with public subnets having internet gateway routes and private subnets remaining isolated from direct internet access."

**Closing trigger**
Always create subnets in multiple AZs for high availability.

---

## Security Groups

**Primary Trigger:** How do you control what network traffic can reach your EC2 instances?

**What it is**
- Virtual firewall rules at instance level
- Control inbound and outbound traffic
- Stateful: return traffic automatically allowed

**Why / When used**
- Allow SSH/RDP access from specific IPs
- Permit HTTP/HTTPS traffic to web servers
- Restrict database access to application tier only
- Default deny, must explicitly allow traffic

**Key constructs**
- `aws_security_group`
- `ingress` rules (inbound)
- `egress` rules (outbound)
- `from_port`, `to_port`, `protocol`
- `cidr_blocks` or `security_groups` as source

**Minimal Terraform example**

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

# Security group for web servers
resource "aws_security_group" "web" {
  name        = "web-sg"
  description = "Security group for web servers"
  vpc_id      = aws_vpc.main.id
  
  # Allow HTTP from anywhere
  ingress {
    description = "HTTP from internet"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  # Allow HTTPS from anywhere
  ingress {
    description = "HTTPS from internet"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  # Allow SSH from company office only
  ingress {
    description = "SSH from office"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["203.0.113.0/24"]  # Company IP range
  }
  
  # Allow all outbound traffic
  egress {
    description = "All outbound traffic"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = {
    Name = "Web Security Group"
  }
}

# Security group for application servers
resource "aws_security_group" "app" {
  name        = "app-sg"
  description = "Security group for application servers"
  vpc_id      = aws_vpc.main.id
  
  # Allow traffic from web tier only
  ingress {
    description     = "HTTP from web tier"
    from_port       = 8080
    to_port         = 8080
    protocol        = "tcp"
    security_groups = [aws_security_group.web.id]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = {
    Name = "App Security Group"
  }
}

# Security group for database
resource "aws_security_group" "database" {
  name        = "db-sg"
  description = "Security group for RDS database"
  vpc_id      = aws_vpc.main.id
  
  # Allow MySQL from app tier only
  ingress {
    description     = "MySQL from app tier"
    from_port       = 3306
    to_port         = 3306
    protocol        = "tcp"
    security_groups = [aws_security_group.app.id]
  }
  
  # No outbound rules needed for RDS
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = {
    Name = "Database Security Group"
  }
}
```

**Example scenario**
- Three-tier architecture: web, app, database
- Web security group allows HTTP/HTTPS from internet
- App security group allows traffic only from web security group
- Database security group allows MySQL only from app security group
- Each tier isolated, minimal attack surface

**KT / interview line**
"Security groups act as virtual firewalls controlling inbound and outbound traffic to AWS resources, with rules specifying allowed ports, protocols, and source IPs or security groups."

**Closing trigger**
Always use specific CIDR blocks or security group references, never 0.0.0.0/0 for sensitive ports.

---

## EC2 Instances

**Primary Trigger:** How do you create virtual machines in AWS with Terraform?

**What it is**
- Elastic Compute Cloud, virtual servers in AWS
- Configurable CPU, memory, storage, networking
- Launch from Amazon Machine Images (AMIs)

**Why / When used**
- Run application servers, web servers, batch jobs
- Scale compute capacity up or down
- Choose instance types based on workload requirements
- Use user_data for automated configuration

**Key constructs**
- `aws_instance`
- `ami` (machine image ID)
- `instance_type` (t2.micro, t3.medium, etc.)
- `subnet_id`, `vpc_security_group_ids`
- `user_data` for bootstrap scripts
- `key_name` for SSH access

**Minimal Terraform example**

```hcl
provider "aws" {
  region = "us-east-1"
}

# Data source for latest Amazon Linux 2 AMI
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = true
}

resource "aws_security_group" "web" {
  name   = "web-sg"
  vpc_id = aws_vpc.main.id
  
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
}

# Basic EC2 instance
resource "aws_instance" "web" {
  ami                    = data.aws_ami.amazon_linux.id
  instance_type          = "t2.micro"
  subnet_id              = aws_subnet.public.id
  vpc_security_group_ids = [aws_security_group.web.id]
  
  # Bootstrap script
  user_data = <<-EOF
              #!/bin/bash
              yum update -y
              yum install -y httpd
              systemctl start httpd
              systemctl enable httpd
              echo "<h1>Hello from Terraform</h1>" > /var/www/html/index.html
              EOF
  
  # EBS root volume configuration
  root_block_device {
    volume_size           = 20
    volume_type           = "gp3"
    delete_on_termination = true
  }
  
  tags = {
    Name = "WebServer"
  }
}

# Multiple instances with count
resource "aws_instance" "app" {
  count                  = 3
  ami                    = data.aws_ami.amazon_linux.id
  instance_type          = "t2.micro"
  subnet_id              = aws_subnet.public.id
  vpc_security_group_ids = [aws_security_group.web.id]
  
  tags = {
    Name = "AppServer-${count.index + 1}"
  }
}

# Outputs
output "web_public_ip" {
  value = aws_instance.web.public_ip
}

output "web_public_dns" {
  value = aws_instance.web.public_dns
}

output "app_private_ips" {
  value = aws_instance.app[*].private_ip
}
```

**Common instance types:**
```
t2.micro   - 1 vCPU, 1 GB RAM   (free tier)
t2.small   - 1 vCPU, 2 GB RAM
t3.medium  - 2 vCPU, 4 GB RAM
t3.large   - 2 vCPU, 8 GB RAM
m5.xlarge  - 4 vCPU, 16 GB RAM  (general purpose)
c5.xlarge  - 4 vCPU, 8 GB RAM   (compute optimized)
r5.xlarge  - 4 vCPU, 32 GB RAM  (memory optimized)
```

**Example scenario**
- Deploy web server running Apache
- Use data source to get latest Amazon Linux AMI
- Instance launches in public subnet with security group
- User_data script installs and starts Apache automatically
- Terraform outputs public IP for accessing the server

**KT / interview line**
"EC2 instances are virtual machines created with specified AMI, instance type, networking, and security groups, with user_data enabling automated configuration during launch."

**Closing trigger**
Always use data sources for AMIs to get latest images automatically.

---

## AMI Data Source

**Primary Trigger:** How do you avoid hardcoding AMI IDs that change per region?

**What it is**
- Query to find Amazon Machine Image IDs dynamically
- Filter by name, owner, tags, or other attributes
- Returns latest matching AMI

**Why / When used**
- AMI IDs differ across AWS regions
- AWS updates AMIs regularly with security patches
- Avoid manual updates when new AMIs released
- Ensure consistent image selection across environments

**Key constructs**
- `data "aws_ami"`
- `most_recent` flag
- `owners` (AWS account IDs or aliases)
- `filter` blocks for search criteria
- `name_regex` for pattern matching

**Minimal Terraform example**

```hcl
provider "aws" {
  region = "us-east-1"
}

# Latest Amazon Linux 2
data "aws_ami" "amazon_linux_2" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
  
  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

# Latest Ubuntu 22.04
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]  # Canonical's AWS account
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}

# Windows Server 2022
data "aws_ami" "windows" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["Windows_Server-2022-English-Full-Base-*"]
  }
  
  filter {
    name   = "platform"
    values = ["windows"]
  }
}

# Custom AMI with tag
data "aws_ami" "custom" {
  most_recent = true
  owners      = ["self"]
  
  filter {
    name   = "tag:Environment"
    values = ["production"]
  }
  
  filter {
    name   = "tag:Application"
    values = ["webapp"]
  }
}

# Use AMI in instance
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux_2.id
  instance_type = "t2.micro"
  
  tags = {
    Name    = "WebServer"
    AMI_ID  = data.aws_ami.amazon_linux_2.id
    AMI_Name = data.aws_ami.amazon_linux_2.name
  }
}

# Outputs to verify AMI details
output "selected_ami_id" {
  value = data.aws_ami.amazon_linux_2.id
}

output "selected_ami_name" {
  value = data.aws_ami.amazon_linux_2.name
}

output "selected_ami_creation_date" {
  value = data.aws_ami.amazon_linux_2.creation_date
}
```

**Common AMI owners:**
```
amazon           - AWS official AMIs
099720109477     - Canonical (Ubuntu)
679593333241     - CentOS
self             - Your AWS account
```

**Example scenario**
- You deploy infrastructure in multiple regions
- Each region has different AMI IDs for same OS
- Use data source with filters to find latest Amazon Linux
- Code works in any region without changes
- Automatically gets updated when AWS releases new AMI

**KT / interview line**
"AMI data source dynamically queries AWS for machine image IDs using filters and most_recent flag, avoiding hardcoded IDs and automatically selecting latest images across regions."

**Closing trigger**
Always use data sources instead of hardcoding AMI IDs.

---

## Key Pairs

**Primary Trigger:** How do you enable SSH access to EC2 instances with Terraform?

**What it is**
- SSH public key registered in AWS
- Used for authenticating to EC2 instances
- Private key stays with you, public key in AWS

**Why / When used**
- Required for SSH access to Linux instances
- Required for RDP decryption on Windows instances
- Each region needs its own key pair registration
- Can import existing keys or let AWS generate

**Key constructs**
- `aws_key_pair`
- `key_name` attribute
- `public_key` content
- Reference in `aws_instance`

**Minimal Terraform example**

```hcl
provider "aws" {
  region = "us-east-1"
}

# Generate key locally first:
# ssh-keygen -t rsa -b 4096 -f ~/.ssh/terraform-aws -N ""

# Import existing public key
resource "aws_key_pair" "deployer" {
  key_name   = "deployer-key"
  public_key = file("~/.ssh/terraform-aws.pub")
  
  tags = {
    Name = "Deployer Key"
  }
}

# Or paste public key directly
resource "aws_key_pair" "admin" {
  key_name   = "admin-key"
  public_key = "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC... admin@example.com"
  
  tags = {
    Name = "Admin Key"
  }
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = true
}

resource "aws_security_group" "ssh" {
  name   = "ssh-sg"
  vpc_id = aws_vpc.main.id
  
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # Restrict to your IP in production
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# EC2 instance with key pair
resource "aws_instance" "web" {
  ami                    = "ami-0c55b159cbfafe1f0"
  instance_type          = "t2.micro"
  key_name               = aws_key_pair.deployer.key_name
  subnet_id              = aws_subnet.public.id
  vpc_security_group_ids = [aws_security_group.ssh.id]
  
  tags = {
    Name = "WebServer"
  }
}

# Outputs for connection
output "instance_public_ip" {
  value = aws_instance.web.public_ip
}

output "ssh_command" {
  value = "ssh -i ~/.ssh/terraform-aws ec2-user@${aws_instance.web.public_ip}"
}
```

**Generate and use key pair:**
```bash
# Generate SSH key pair locally
ssh-keygen -t rsa -b 4096 -f ~/.ssh/terraform-aws -N ""

# Apply Terraform configuration
terraform apply

# SSH to instance
ssh -i ~/.ssh/terraform-aws ec2-user@<instance_public_ip>

# Or use output
$(terraform output -raw ssh_command)
```

**Example scenario**
- You need to SSH into EC2 instances for management
- Generate SSH key pair on your local machine
- Create aws_key_pair resource with public key content
- Reference key_name in EC2 instance
- SSH using private key from your machine

**KT / interview line**
"Key pairs register SSH public keys in AWS for EC2 authentication, allowing SSH access using the corresponding private key while keeping private keys secure on local machines."

**Closing trigger**
Never commit private keys to version control, only public keys.

---

## Application Load Balancer

**Primary Trigger:** How do you distribute incoming traffic across multiple EC2 instances?

**What it is**
- Layer 7 load balancer operating at HTTP/HTTPS level
- Distributes traffic to healthy targets across AZs
- Supports path-based and host-based routing

**Why / When used**
- Distribute traffic across multiple EC2 instances
- Achieve high availability and fault tolerance
- Handle SSL/TLS termination at load balancer
- Route different URLs to different backend services

**Key constructs**
- `aws_lb` (application type)
- `aws_lb_target_group`
- `aws_lb_listener`
- `aws_lb_target_group_attachment`
- Health checks

**Minimal Terraform example**

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public_1" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-east-1a"
}

resource "aws_subnet" "public_2" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.2.0/24"
  availability_zone = "us-east-1b"
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
}

resource "aws_security_group" "alb" {
  name   = "alb-sg"
  vpc_id = aws_vpc.main.id
  
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_security_group" "web" {
  name   = "web-sg"
  vpc_id = aws_vpc.main.id
  
  ingress {
    from_port       = 80
    to_port         = 80
    protocol        = "tcp"
    security_groups = [aws_security_group.alb.id]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Application Load Balancer
resource "aws_lb" "web" {
  name               = "web-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets            = [aws_subnet.public_1.id, aws_subnet.public_2.id]
  
  tags = {
    Name = "Web ALB"
  }
}

# Target group
resource "aws_lb_target_group" "web" {
  name     = "web-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id
  
  health_check {
    enabled             = true
    healthy_threshold   = 2
    unhealthy_threshold = 2
    timeout             = 5
    interval            = 30
    path                = "/"
    matcher             = "200"
  }
  
  tags = {
    Name = "Web Target Group"
  }
}

# Listener
resource "aws_lb_listener" "web" {
  load_balancer_arn = aws_lb.web.arn
  port              = 80
  protocol          = "HTTP"
  
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.web.arn
  }
}

# EC2 instances (simplified)
resource "aws_instance" "web" {
  count                  = 2
  ami                    = "ami-0c55b159cbfafe1f0"
  instance_type          = "t2.micro"
  subnet_id              = element([aws_subnet.public_1.id, aws_subnet.public_2.id], count.index)
  vpc_security_group_ids = [aws_security_group.web.id]
  
  user_data = <<-EOF
              #!/bin/bash
              yum update -y
              yum install -y httpd
              echo "Server ${count.index + 1}" > /var/www/html/index.html
              systemctl start httpd
              EOF
  
  tags = {
    Name = "WebServer-${count.index + 1}"
  }
}

# Attach instances to target group
resource "aws_lb_target_group_attachment" "web" {
  count            = 2
  target_group_arn = aws_lb_target_group.web.arn
  target_id        = aws_instance.web[count.index].id
  port             = 80
}

# Output
output "alb_dns_name" {
  value = aws_lb.web.dns_name
}
```

**Example scenario**
- You have multiple web servers for high availability
- Create ALB in two public subnets across different AZs
- Configure target group with health checks
- Attach EC2 instances to target group
- Traffic distributed automatically to healthy instances

**KT / interview line**
"Application Load Balancer distributes HTTP/HTTPS traffic across multiple targets in different availability zones, performing health checks and routing only to healthy instances."

**Closing trigger**
Always deploy ALB across multiple AZs for high availability.

---

## Auto Scaling Group

**Primary Trigger:** How do you automatically scale EC2 instances based on demand?

**What it is**
- Automatically adjusts number of EC2 instances
- Maintains desired capacity between min and max
- Scales based on CloudWatch metrics or schedules

**Why / When used**
- Handle variable traffic loads automatically
- Maintain high availability by replacing unhealthy instances
- Optimize costs by scaling down during low traffic
- Integrate with ALB for distributed load

**Key constructs**
- `aws_autoscaling_group`
- `aws_launch_template`
- `min_size`, `max_size`, `desired_capacity`
- `health_check_type`
- `target_group_arns` for ALB integration

**Minimal Terraform example**

```hcl
provider "aws" {
  region = "us-east-1"
}

data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public_1" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-east-1a"
}

resource "aws_subnet" "public_2" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.2.0/24"
  availability_zone = "us-east-1b"
}

resource "aws_security_group" "web" {
  name   = "web-sg"
  vpc_id = aws_vpc.main.id
  
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Launch template defines instance configuration
resource "aws_launch_template" "web" {
  name_prefix   = "web-lt-"
  image_id      = data.aws_ami.amazon_linux.id
  instance_type = "t2.micro"
  
  vpc_security_group_ids = [aws_security_group.web.id]
  
  user_data = base64encode(<<-EOF
              #!/bin/bash
              yum update -y
              yum install -y httpd
              echo "Served by $(hostname)" > /var/www/html/index.html
              systemctl start httpd
              systemctl enable httpd
              EOF
  )
  
  tag_specifications {
    resource_type = "instance"
    
    tags = {
      Name = "ASG-WebServer"
    }
  }
}

# Auto Scaling Group
resource "aws_autoscaling_group" "web" {
  name                = "web-asg"
  vpc_zone_identifier = [aws_subnet.public_1.id, aws_subnet.public_2.id]
  
  min_size         = 2
  max_size         = 6
  desired_capacity = 2
  
  health_check_type         = "EC2"
  health_check_grace_period = 300
  
  launch_template {
    id      = aws_launch_template.web.id
    version = "$Latest"
  }
  
  tag {
    key                 = "Name"
    value               = "ASG-WebServer"
    propagate_at_launch = true
  }
  
  tag {
    key                 = "Environment"
    value               = "Production"
    propagate_at_launch = true
  }
}

# Scaling policy - scale up
resource "aws_autoscaling_policy" "scale_up" {
  name                   = "scale-up"
  scaling_adjustment     = 1
  adjustment_type        = "ChangeInCapacity"
  cooldown               = 300
  autoscaling_group_name = aws_autoscaling_group.web.name
}

# Scaling policy - scale down
resource "aws_autoscaling_policy" "scale_down" {
  name                   = "scale-down"
  scaling_adjustment     = -1
  adjustment_type        = "ChangeInCapacity"
  cooldown               = 300
  autoscaling_group_name = aws_autoscaling_group.web.name
}

# CloudWatch alarm - high CPU
resource "aws_cloudwatch_metric_alarm" "high_cpu" {
  alarm_name          = "high-cpu"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = 120
  statistic           = "Average"
  threshold           = 70
  
  dimensions = {
    AutoScalingGroupName = aws_autoscaling_group.web.name
  }
  
  alarm_actions = [aws_autoscaling_policy.scale_up.arn]
}

# CloudWatch alarm - low CPU
resource "aws_cloudwatch_metric_alarm" "low_cpu" {
  alarm_name          = "low-cpu"
  comparison_operator = "LessThanThreshold"
  evaluation_periods  = 2
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = 120
  statistic           = "Average"
  threshold           = 20
  
  dimensions = {
    AutoScalingGroupName = aws_autoscaling_group.web.name
  }
  
  alarm_actions = [aws_autoscaling_policy.scale_down.arn]
}
```

**Example scenario**
- Your web application has unpredictable traffic patterns
- Configure ASG with min 2, max 6, desired 2 instances
- When CPU exceeds 70%, ASG launches additional instances
- When CPU drops below 20%, ASG terminates excess instances
- Always maintains at least 2 instances for high availability

**KT / interview line**
"Auto Scaling Group maintains specified number of EC2 instances using launch templates, automatically replacing unhealthy instances and scaling capacity based on CloudWatch alarms or schedules."

**Closing trigger**
Always set min capacity to ensure high availability.

---

## S3 Buckets

**Primary Trigger:** How do you create object storage buckets in AWS with Terraform?

**What it is**
- Simple Storage Service for object storage
- Store files, backups, logs, static website content
- Globally unique bucket names

**Why / When used**
- Store application assets and user uploads
- Host static websites
- Store Terraform state files remotely
- Archive logs and backups
- Data lake storage for analytics

**Key constructs**
- `aws_s3_bucket`
- `aws_s3_bucket_versioning`
- `aws_s3_bucket_public_access_block`
- `aws_s3_bucket_policy`
- Bucket must have globally unique name

**Minimal Terraform example**

```hcl
provider "aws" {
  region = "us-east-1"
}

# Basic S3 bucket
resource "aws_s3_bucket" "app_data" {
  bucket = "my-app-data-bucket-12345"  # Must be globally unique
  
  tags = {
    Name        = "App Data Bucket"
    Environment = "Production"
  }
}

# Enable versioning
resource "aws_s3_bucket_versioning" "app_data" {
  bucket = aws_s3_bucket.app_data.id
  
  versioning_configuration {
    status = "Enabled"
  }
}

# Block public access (security best practice)
resource "aws_s3_bucket_public_access_block" "app_data" {
  bucket = aws_s3_bucket.app_data.id
  
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# S3 bucket for static website
resource "aws_s3_bucket" "website" {
  bucket = "my-static-website-12345"
  
  tags = {
    Name = "Static Website Bucket"
  }
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

# Bucket policy for public read (website)
resource "aws_s3_bucket_policy" "website" {
  bucket = aws_s3_bucket.website.id
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Sid       = "PublicReadGetObject"
      Effect    = "Allow"
      Principal = "*"
      Action    = "s3:GetObject"
      Resource  = "${aws_s3_bucket.website.arn}/*"
    }]
  })
}

# S3 bucket for Terraform state
resource "aws_s3_bucket" "terraform_state" {
  bucket = "company-terraform-state-12345"
  
  tags = {
    Name = "Terraform State Bucket"
  }
}

resource "aws_s3_bucket_versioning" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

# S3 bucket for logs
resource "aws_s3_bucket" "logs" {
  bucket = "my-app-logs-12345"
  
  tags = {
    Name = "Application Logs"
  }
}

resource "aws_s3_bucket_lifecycle_configuration" "logs" {
  bucket = aws_s3_bucket.logs.id
  
  rule {
    id     = "delete-old-logs"
    status = "Enabled"
    
    expiration {
      days = 90
    }
  }
  
  rule {
    id     = "transition-to-glacier"
    status = "Enabled"
    
    transition {
      days          = 30
      storage_class = "GLACIER"
    }
  }
}

# Outputs
output "app_data_bucket_name" {
  value = aws_s3_bucket.app_data.id
}

output "website_endpoint" {
  value = aws_s3_bucket_website_configuration.website.website_endpoint
}
```

**Example scenario**
- You need to store user-uploaded files
- Create S3 bucket with versioning enabled
- Block public access for security
- EC2 instances access bucket using IAM role
- Lifecycle rules delete old files automatically

**KT / interview line**
"S3 buckets provide object storage with globally unique names, supporting features like versioning, encryption, lifecycle policies, and public or private access control."

**Closing trigger**
Always block public access unless explicitly hosting public content.

---

## RDS

**Primary Trigger:** How do you create managed relational databases in AWS with Terraform?

**What it is**
- Relational Database Service, managed SQL databases
- Supports MySQL, PostgreSQL, Oracle, SQL Server, MariaDB
- AWS handles backups, patching, and high availability

**Why / When used**
- Run production databases without managing servers
- Automatic backups and point-in-time recovery
- Multi-AZ deployments for high availability
- Read replicas for scaling read operations

**Key constructs**
- `aws_db_instance`
- `engine` (mysql, postgres, etc.)
- `instance_class` (size)
- `allocated_storage`
- `db_subnet_group` for networking
- Master username and password

**Minimal Terraform example**

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "private_1" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-east-1a"
}

resource "aws_subnet" "private_2" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.2.0/24"
  availability_zone = "us-east-1b"
}

# DB subnet group (requires at least 2 subnets in different AZs)
resource "aws_db_subnet_group" "main" {
  name       = "main-db-subnet-group"
  subnet_ids = [aws_subnet.private_1.id, aws_subnet.private_2.id]
  
  tags = {
    Name = "Main DB Subnet Group"
  }
}

# Security group for RDS
resource "aws_security_group" "rds" {
  name   = "rds-sg"
  vpc_id = aws_vpc.main.id
  
  ingress {
    from_port   = 3306
    to_port     = 3306
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/16"]  # Allow from VPC only
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = {
    Name = "RDS Security Group"
  }
}

# RDS MySQL instance
resource "aws_db_instance" "main" {
  identifier     = "myapp-db"
  engine         = "mysql"
  engine_version = "8.0"
  
  instance_class    = "db.t3.micro"
  allocated_storage = 20
  storage_type      = "gp3"
  storage_encrypted = true
  
  db_name  = "myappdb"
  username = "admin"
  password = "changeme123456"  # Use AWS Secrets Manager in production
  
  db_subnet_group_name   = aws_db_subnet_group.main.name
  vpc_security_group_ids = [aws_security_group.rds.id]
  
  # Backup configuration
  backup_retention_period = 7
  backup_window           = "03:00-04:00"
  maintenance_window      = "mon:04:00-mon:05:00"
  
  # High availability (creates standby in different AZ)
  multi_az = false  # Set true for production
  
  # Prevent accidental deletion
  deletion_protection = true
  skip_final_snapshot = false
  final_snapshot_identifier = "myapp-db-final-snapshot"
  
  # Performance Insights
  enabled_cloudwatch_logs_exports = ["error", "slowquery"]
  
  tags = {
    Name        = "MyApp Database"
    Environment = "Production"
  }
}

# PostgreSQL example
resource "aws_db_instance" "postgres" {
  identifier     = "myapp-postgres"
  engine         = "postgres"
  engine_version = "15"
  
  instance_class    = "db.t3.micro"
  allocated_storage = 20
  
  db_name  = "myappdb"
  username = "postgres"
  password = "changeme123456"
  
  db_subnet_group_name   = aws_db_subnet_group.main.name
  vpc_security_group_ids = [aws_security_group.rds.id]
  
  skip_final_snapshot = true  # For testing only
  
  tags = {
    Name = "PostgreSQL Database"
  }
}

# Outputs
output "rds_endpoint" {
  description = "Database connection endpoint"
  value       = aws_db_instance.main.endpoint
}

output "rds_address" {
  description = "Database hostname"
  value       = aws_db_instance.main.address
}

output "rds_port" {
  description = "Database port"
  value       = aws_db_instance.main.port
}
```

**Common instance classes:**
```
db.t3.micro   - 2 vCPU, 1 GB RAM   (dev/test)
db.t3.small   - 2 vCPU, 2 GB RAM
db.t3.medium  - 2 vCPU, 4 GB RAM
db.r5.large   - 2 vCPU, 16 GB RAM  (memory optimized)
db.r5.xlarge  - 4 vCPU, 32 GB RAM
```

**Example scenario**
- Your application needs MySQL database
- Create RDS instance in private subnets
- Configure security group allowing access from app tier only
- Enable automated backups with 7-day retention
- Use db_subnet_group spanning multiple AZs for Multi-AZ option

**KT / interview line**
"RDS provides managed relational databases with automatic backups, patching, and optional Multi-AZ high availability, requiring db_subnet_group and security groups for network configuration."

**Closing trigger**
Always enable deletion_protection and backups for production databases.

---

## Tags

**Primary Trigger:** How do you organize and track AWS resources across teams and environments?

**What it is**
- Key-value pairs attached to AWS resources
- Metadata for organization, billing, automation
- Consistent tagging enables governance

**Why / When used**
- Track costs by environment, project, or team
- Automate operations based on tags
- Enforce compliance and security policies
- Identify resource ownership
- Required for cost allocation and chargeback

**Key constructs**
- `tags = { }` map in resource blocks
- `default_tags` in provider configuration
- Merge function for combining tag sets
- Tag policies for governance

**Minimal Terraform example**

```hcl
provider "aws" {
  region = "us-east-1"
  
  # Default tags applied to all resources
  default_tags {
    tags = {
      ManagedBy   = "Terraform"
      CostCenter  = "Engineering"
      Compliance  = "HIPAA"
    }
  }
}

# Variables for common tags
variable "environment" {
  type    = string
  default = "production"
}

variable "project" {
  type    = string
  default = "webapp"
}

# Locals for tag standardization
locals {
  common_tags = {
    Environment = var.environment
    Project     = var.project
    Owner       = "DevOps Team"
    Terraform   = "true"
  }
  
  name_prefix = "${var.project}-${var.environment}"
}

# VPC with merged tags
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-vpc"
    Type = "Network"
  })
}

# EC2 instances with specific tags
resource "aws_instance" "web" {
  count         = 3
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = merge(local.common_tags, {
    Name        = "${local.name_prefix}-web-${count.index + 1}"
    Role        = "WebServer"
    Backup      = "Daily"
    Monitoring  = "Enabled"
  })
  
  volume_tags = merge(local.common_tags, {
    Name       = "${local.name_prefix}-web-${count.index + 1}-root"
    VolumeType = "Root"
  })
}

# S3 bucket with compliance tags
resource "aws_s3_bucket" "data" {
  bucket = "myapp-data-${var.environment}"
  
  tags = merge(local.common_tags, {
    Name           = "${local.name_prefix}-data-bucket"
    DataClass      = "Confidential"
    Retention      = "7-years"
    Encryption     = "Required"
  })
}

# RDS with cost tracking tags
resource "aws_db_instance" "main" {
  identifier     = "${local.name_prefix}-db"
  engine         = "mysql"
  instance_class = "db.t3.micro"
  
  allocated_storage = 20
  username          = "admin"
  password          = "changeme123"
  
  skip_final_snapshot = true
  
  tags = merge(local.common_tags, {
    Name         = "${local.name_prefix}-database"
    Database     = "MySQL"
    BackupPolicy = "7-day-retention"
    ChargeCode   = "PROJ-001"
  })
}

# Security group with automation tags
resource "aws_security_group" "app" {
  name   = "${local.name_prefix}-app-sg"
  vpc_id = aws_vpc.main.id
  
  tags = merge(local.common_tags, {
    Name           = "${local.name_prefix}-app-sg"
    AutoShutdown   = "true"
    ShutdownTime   = "20:00"
    StartupTime    = "08:00"
  })
}
```

**Tagging best practices example:**
```hcl
# Standard organizational tags
{
  # Identity
  Name        = "Resource descriptive name"
  Environment = "dev|staging|prod"
  Project     = "Project code or name"
  Owner       = "Team or individual"
  
  # Operations
  ManagedBy   = "Terraform"
  Backup      = "Daily|Weekly|None"
  Monitoring  = "Enabled|Disabled"
  
  # Cost Management
  CostCenter  = "Department code"
  ChargeCode  = "Project billing code"
  
  # Compliance
  DataClass   = "Public|Internal|Confidential|Restricted"
  Compliance  = "HIPAA|PCI|SOC2"
  
  # Lifecycle
  CreatedBy   = "User or system"
  CreatedDate = "YYYY-MM-DD"
  ExpiryDate  = "YYYY-MM-DD"
}
```

**Example scenario**
- Your company has multiple teams and projects
- Configure default_tags in provider for organization-wide tags
- Use locals for environment-specific common tags
- Each resource gets Name, Environment, Project tags
- Finance uses CostCenter tags for chargeback
- Automation uses tags to stop/start non-production instances

**KT / interview line**
"Tags are key-value metadata attached to AWS resources for organization, cost tracking, and automation, applied using tags blocks, default_tags in provider, or merge function for combining tag sets."

**Closing trigger**
Establish tagging standards early and enforce them consistently.

---

**END OF DOCUMENT**

This cheat sheet covers all major Terraform and AWS topics with the exact structure you requested. Each section includes the trigger, explanation, real-world context, code examples, and knowledge transfer lines for effective retrieval and teaching.
