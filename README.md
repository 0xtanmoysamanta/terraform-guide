# terraform-guide
A brief terraform guide 
# Terraform Guide: Beginner to Intermediate

## Table of Contents
1. [What is Terraform?](#what-is-terraform)
2. [Why Use Terraform?](#why-use-terraform)
3. [Core Concepts](#core-concepts)
4. [Installation](#installation)
5. [Basic Workflow](#basic-workflow)
6. [HCL Syntax Basics](#hcl-syntax-basics)
7. [Providers](#providers)
8. [Resources](#resources)
9. [Variables](#variables)
10. [Outputs](#outputs)
11. [Data Sources](#data-sources)
12. [State Management](#state-management)
13. [Modules](#modules)
14. [Workspaces](#workspaces)
15. [Best Practices](#best-practices)
16. [Common Patterns](#common-patterns)
17. [Real-World Examples](#real-world-examples)

---

## What is Terraform?

Terraform is an Infrastructure as Code (IaC) tool created by HashiCorp. It allows you to define and provision infrastructure using declarative configuration files. You write code to describe your desired infrastructure state, and Terraform makes it happen.

Think of Terraform like a blueprint for your cloud infrastructure - you describe what you want, and Terraform figures out how to build it.

---

## Why Use Terraform?

- **Infrastructure as Code**: Version control your infrastructure
- **Multi-Cloud**: Works with AWS, Azure, GCP, and 1000+ providers
- **Declarative**: Describe the desired state, not the steps
- **Plan Before Apply**: Preview changes before execution
- **Dependency Management**: Automatically handles resource dependencies
- **Reusable**: Create modules for repeatable infrastructure patterns
- **State Management**: Tracks infrastructure and detects drift

---

## Core Concepts

### Infrastructure as Code (IaC)
Managing infrastructure through code rather than manual processes.

### Providers
Plugins that interact with cloud platforms and APIs (AWS, Azure, GCP, etc.)

### Resources
Infrastructure components you want to create (EC2 instances, S3 buckets, etc.)

### State
Terraform's record of your infrastructure, stored in a state file.

### Plan
A preview of changes Terraform will make to reach the desired state.

### Modules
Reusable Terraform configurations that can be shared and versioned.

### Variables
Input parameters to make configurations flexible and reusable.

### Outputs
Values exported from your Terraform configuration.

---

## Installation

### Linux
```bash
wget https://releases.hashicorp.com/terraform/1.6.0/terraform_1.6.0_linux_amd64.zip
unzip terraform_1.6.0_linux_amd64.zip
sudo mv terraform /usr/local/bin/
```

### macOS
```bash
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
```

### Windows
Download from [terraform.io](https://www.terraform.io/downloads) or use Chocolatey:
```powershell
choco install terraform
```

### Verify Installation
```bash
terraform version
terraform -help
```

---

## Basic Workflow

The core Terraform workflow consists of:

```bash
# 1. Initialize - Download provider plugins
terraform init

# 2. Format - Format your configuration files
terraform fmt

# 3. Validate - Check configuration syntax
terraform validate

# 4. Plan - Preview changes
terraform plan

# 5. Apply - Create/update infrastructure
terraform apply

# 6. Destroy - Remove infrastructure
terraform destroy
```

### Workflow Diagram
```
Write Config → Init → Plan → Apply → State Updated
                ↑                        ↓
                └────── Modify ──────────┘
```

---

## HCL Syntax Basics

Terraform uses HashiCorp Configuration Language (HCL).

### Basic Structure
```hcl
# Comments start with #

// Or double slashes

/* Multi-line
   comments */

# Block syntax
block_type "label" "name" {
  argument = "value"
  nested_block {
    argument = value
  }
}
```

### Data Types
```hcl
# String
name = "example"

# Number
count = 3
port = 8080

# Boolean
enabled = true

# List
availability_zones = ["us-east-1a", "us-east-1b"]

# Map
tags = {
  Environment = "production"
  Team        = "devops"
}

# Object
instance = {
  ami           = "ami-12345678"
  instance_type = "t2.micro"
}
```

### Expressions
```hcl
# String interpolation
name = "server-${var.environment}"

# Arithmetic
total = var.count * 2

# Comparison
condition = var.count > 5

# Conditional
instance_type = var.environment == "prod" ? "t3.large" : "t3.micro"

# Functions
upper_name = upper(var.name)
timestamp  = timestamp()
```

---

## Providers

Providers are plugins that Terraform uses to interact with APIs.

### Basic Provider Configuration
```hcl
# AWS Provider
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  required_version = ">= 1.0"
}

provider "aws" {
  region = "us-east-1"
  
  default_tags {
    tags = {
      Project     = "MyApp"
      Environment = "Production"
    }
  }
}
```

### Multiple Provider Configurations
```hcl
# Default provider
provider "aws" {
  region = "us-east-1"
}

# Alternate provider
provider "aws" {
  alias  = "west"
  region = "us-west-2"
}

# Use alternate provider
resource "aws_instance" "west_server" {
  provider = aws.west
  ami      = "ami-12345678"
  instance_type = "t3.micro"
}
```

### Common Providers
```hcl
# Azure
provider "azurerm" {
  features {}
  subscription_id = var.subscription_id
}

# Google Cloud
provider "google" {
  project = var.project_id
  region  = "us-central1"
}

# Kubernetes
provider "kubernetes" {
  config_path = "~/.kube/config"
}

# Docker
provider "docker" {
  host = "unix:///var/run/docker.sock"
}
```

---

## Resources

Resources are the most important element - they define infrastructure components.

### Basic Resource
```hcl
resource "aws_instance" "web_server" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  
  tags = {
    Name = "WebServer"
  }
}
```

### Resource with Dependencies
```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  
  tags = {
    Name = "main-vpc"
  }
}

resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id  # Implicit dependency
  cidr_block = "10.0.1.0/24"
  
  tags = {
    Name = "public-subnet"
  }
}

resource "aws_instance" "app" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  subnet_id     = aws_subnet.public.id
  
  depends_on = [aws_vpc.main]  # Explicit dependency
  
  tags = {
    Name = "app-server"
  }
}
```

### Resource Meta-Arguments

#### count
```hcl
resource "aws_instance" "server" {
  count         = 3
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  
  tags = {
    Name = "server-${count.index}"
  }
}

# Access: aws_instance.server[0].id
```

#### for_each
```hcl
variable "instances" {
  type = map(object({
    instance_type = string
  }))
  default = {
    web = {
      instance_type = "t3.micro"
    }
    api = {
      instance_type = "t3.small"
    }
  }
}

resource "aws_instance" "server" {
  for_each      = var.instances
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = each.value.instance_type
  
  tags = {
    Name = each.key
  }
}

# Access: aws_instance.server["web"].id
```

#### lifecycle
```hcl
resource "aws_instance" "app" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  
  lifecycle {
    create_before_destroy = true
    prevent_destroy       = false
    ignore_changes        = [tags]
  }
}
```

---

## Variables

Variables make configurations flexible and reusable.

### Variable Declaration
```hcl
# variables.tf

variable "environment" {
  description = "Environment name"
  type        = string
  default     = "dev"
}

variable "instance_count" {
  description = "Number of instances"
  type        = number
  default     = 1
}

variable "enable_monitoring" {
  description = "Enable CloudWatch monitoring"
  type        = bool
  default     = false
}

variable "availability_zones" {
  description = "List of AZs"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b"]
}

variable "tags" {
  description = "Resource tags"
  type        = map(string)
  default     = {
    Project = "MyApp"
  }
}

variable "instance_config" {
  description = "Instance configuration"
  type = object({
    ami           = string
    instance_type = string
    disk_size     = number
  })
  default = {
    ami           = "ami-0c55b159cbfafe1f0"
    instance_type = "t3.micro"
    disk_size     = 20
  }
}
```

### Variable Validation
```hcl
variable "environment" {
  type        = string
  description = "Environment name"
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "instance_type" {
  type = string
  
  validation {
    condition     = can(regex("^t3\\.", var.instance_type))
    error_message = "Only t3 instance types are allowed."
  }
}
```

### Using Variables
```hcl
# main.tf
resource "aws_instance" "app" {
  ami           = var.instance_config.ami
  instance_type = var.instance_config.instance_type
  
  tags = merge(
    var.tags,
    {
      Name        = "app-${var.environment}"
      Environment = var.environment
    }
  )
}
```

### Providing Variable Values

#### Command Line
```bash
terraform apply -var="environment=prod" -var="instance_count=3"
```

#### Variable File (terraform.tfvars)
```hcl
environment    = "production"
instance_count = 5
tags = {
  Project = "MyApp"
  Team    = "DevOps"
}
```

#### Environment Variables
```bash
export TF_VAR_environment="prod"
export TF_VAR_instance_count=3
terraform apply
```

#### Auto-loaded Files
- `terraform.tfvars`
- `terraform.tfvars.json`
- `*.auto.tfvars`
- `*.auto.tfvars.json`

---

## Outputs

Outputs export values from your configuration.

### Basic Outputs
```hcl
# outputs.tf

output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.app.id
}

output "instance_public_ip" {
  description = "Public IP of the instance"
  value       = aws_instance.app.public_ip
}

output "instance_private_ip" {
  description = "Private IP of the instance"
  value       = aws_instance.app.private_ip
  sensitive   = true
}
```

### Complex Outputs
```hcl
output "instance_details" {
  description = "Instance details"
  value = {
    id         = aws_instance.app.id
    public_ip  = aws_instance.app.public_ip
    private_ip = aws_instance.app.private_ip
    az         = aws_instance.app.availability_zone
  }
}

output "all_instance_ids" {
  description = "IDs of all instances"
  value       = [for i in aws_instance.server : i.id]
}

output "instance_map" {
  description = "Map of instance names to IPs"
  value = {
    for k, v in aws_instance.server : k => v.private_ip
  }
}
```

### Using Outputs
```bash
# View all outputs
terraform output

# View specific output
terraform output instance_id

# Output as JSON
terraform output -json

# Use in scripts
INSTANCE_ID=$(terraform output -raw instance_id)
```

---

## Data Sources

Data sources retrieve information about existing infrastructure.

### Basic Data Source
```hcl
# Get latest Amazon Linux AMI
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

resource "aws_instance" "app" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"
}
```

### Common Data Sources
```hcl
# Availability Zones
data "aws_availability_zones" "available" {
  state = "available"
}

# VPC
data "aws_vpc" "default" {
  default = true
}

# Subnets
data "aws_subnets" "public" {
  filter {
    name   = "vpc-id"
    values = [data.aws_vpc.default.id]
  }
}

# Security Group
data "aws_security_group" "web" {
  name = "web-sg"
}

# Current AWS region
data "aws_region" "current" {}

# Current AWS account
data "aws_caller_identity" "current" {}
```

### Remote State Data Source
```hcl
data "terraform_remote_state" "vpc" {
  backend = "s3"
  config = {
    bucket = "my-terraform-state"
    key    = "vpc/terraform.tfstate"
    region = "us-east-1"
  }
}

resource "aws_instance" "app" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  subnet_id     = data.terraform_remote_state.vpc.outputs.subnet_id
}
```

---

## State Management

Terraform state tracks resource metadata and manages infrastructure.

### Local State (Default)
State stored in `terraform.tfstate` file locally.

```hcl
# terraform.tf
terraform {
  backend "local" {
    path = "terraform.tfstate"
  }
}
```

### Remote State (S3)
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

### Remote State (Terraform Cloud)
```hcl
terraform {
  cloud {
    organization = "my-org"
    workspaces {
      name = "my-workspace"
    }
  }
}
```

### State Commands
```bash
# Show current state
terraform show

# List resources in state
terraform state list

# Show specific resource
terraform state show aws_instance.app

# Move resource in state
terraform state mv aws_instance.old aws_instance.new

# Remove resource from state
terraform state rm aws_instance.app

# Pull remote state
terraform state pull > backup.tfstate

# Push local state to remote
terraform state push backup.tfstate

# Replace provider
terraform state replace-provider hashicorp/aws registry.terraform.io/hashicorp/aws
```

### State Locking
Prevents concurrent modifications.

```hcl
# S3 backend with DynamoDB locking
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

### Import Existing Resources
```bash
# Import existing EC2 instance
terraform import aws_instance.app i-1234567890abcdef0

# First, write the configuration
# main.tf
resource "aws_instance" "app" {
  # Configuration will be populated after import
}

# Then import
terraform import aws_instance.app i-1234567890abcdef0

# View imported state
terraform show aws_instance.app
```

---

## Modules

Modules are containers for multiple resources used together.

### Using a Module
```hcl
module "vpc" {
  source = "./modules/vpc"
  
  cidr_block = "10.0.0.0/16"
  environment = "production"
  
  tags = {
    Project = "MyApp"
  }
}

# Access module outputs
resource "aws_instance" "app" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  subnet_id     = module.vpc.public_subnet_id
}
```

### Creating a Module

#### Directory Structure
```
modules/
└── vpc/
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    └── README.md
```

#### modules/vpc/main.tf
```hcl
resource "aws_vpc" "main" {
  cidr_block           = var.cidr_block
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-vpc"
    }
  )
}

resource "aws_subnet" "public" {
  count                   = length(var.availability_zones)
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.cidr_block, 8, count.index)
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true
  
  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-public-${count.index + 1}"
    }
  )
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-igw"
    }
  )
}
```

#### modules/vpc/variables.tf
```hcl
variable "cidr_block" {
  description = "CIDR block for VPC"
  type        = string
}

variable "environment" {
  description = "Environment name"
  type        = string
}

variable "availability_zones" {
  description = "List of availability zones"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b"]
}

variable "tags" {
  description = "Tags to apply to resources"
  type        = map(string)
  default     = {}
}
```

#### modules/vpc/outputs.tf
```hcl
output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}

output "public_subnet_ids" {
  description = "List of public subnet IDs"
  value       = aws_subnet.public[*].id
}

output "vpc_cidr" {
  description = "VPC CIDR block"
  value       = aws_vpc.main.cidr_block
}
```

### Module Sources
```hcl
# Local path
module "vpc" {
  source = "./modules/vpc"
}

# Terraform Registry
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"
}

# GitHub
module "vpc" {
  source = "github.com/terraform-aws-modules/terraform-aws-vpc"
}

# Git with ref
module "vpc" {
  source = "git::https://github.com/user/repo.git?ref=v1.0.0"
}

# S3 bucket
module "vpc" {
  source = "s3::https://s3.amazonaws.com/bucket/vpc.zip"
}
```

---

## Workspaces

Workspaces allow multiple states for the same configuration.

### Workspace Commands
```bash
# List workspaces
terraform workspace list

# Create new workspace
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod

# Switch workspace
terraform workspace select dev

# Show current workspace
terraform workspace show

# Delete workspace
terraform workspace delete dev
```

### Using Workspaces
```hcl
locals {
  environment = terraform.workspace
  
  instance_type = {
    dev     = "t3.micro"
    staging = "t3.small"
    prod    = "t3.large"
  }
  
  instance_count = {
    dev     = 1
    staging = 2
    prod    = 3
  }
}

resource "aws_instance" "app" {
  count         = local.instance_count[local.environment]
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = local.instance_type[local.environment]
  
  tags = {
    Name        = "app-${local.environment}-${count.index + 1}"
    Environment = local.environment
  }
}
```

---

## Best Practices

### 1. File Organization
```
project/
├── main.tf              # Main resources
├── variables.tf         # Variable declarations
├── outputs.tf           # Output declarations
├── terraform.tf         # Terraform settings and providers
├── terraform.tfvars     # Variable values (not committed)
├── .terraform.lock.hcl  # Provider version lock
├── .gitignore
├── README.md
├── modules/
│   ├── vpc/
│   ├── compute/
│   └── database/
└── environments/
    ├── dev/
    ├── staging/
    └── prod/
```

### 2. Version Control
```.gitignore
# Local .terraform directories
**/.terraform/*

# State files
*.tfstate
*.tfstate.*

# Crash log files
crash.log

# Sensitive variable files
terraform.tfvars
*.auto.tfvars

# Override files
override.tf
override.tf.json

# CLI configuration files
.terraformrc
terraform.rc
```

### 3. Naming Conventions
```hcl
# Use descriptive names
resource "aws_instance" "web_server" { }    # Good
resource "aws_instance" "i1" { }            # Bad

# Use snake_case
resource "aws_s3_bucket" "data_bucket" { }  # Good
resource "aws_s3_bucket" "dataBucket" { }   # Bad

# Use consistent prefixes
variable "app_name" { }
variable "app_environment" { }
variable "app_version" { }
```

### 4. Use Locals for Repeated Values
```hcl
locals {
  common_tags = {
    Project     = "MyApp"
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
  
  name_prefix = "${var.project}-${var.environment}"
}

resource "aws_instance" "app" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  
  tags = merge(
    local.common_tags,
    {
      Name = "${local.name_prefix}-app"
    }
  )
}
```

### 5. Use Data Sources for External Info
```hcl
# Don't hardcode AMI IDs
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]  # Canonical
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
}

resource "aws_instance" "app" {
  ami = data.aws_ami.ubuntu.id
  # ...
}
```

### 6. Use Remote State
```hcl
terraform {
  backend "s3" {
    bucket         = "company-terraform-state"
    key            = "myapp/prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

