# Terraform Standards and Best Practices

## File Organization Standards

### 1. Repository Structure Enforcement

**MANDATORY**: Every repository must have this exact structure:

```
terraform-repository/
├── .modules/          # Custom modules directory
├── dev/              # Development environment
├── staging/          # Staging environment
├── prod/             # Production environment
├── bin/              # Execution scripts directory
│   ├── plan          # Terraform plan script
│   └── apply         # Terraform apply script
└── README.md
```

### 2. Module File Requirements

Each module in `.modules/` directory MUST have:

```hcl
# variables.tf - Input variables
variable "example_variable" {
  description = "Description of the variable"
  type        = string
  default     = "default_value"
}

# main.tf - Resource definitions
resource "aws_example" "main" {
  # Resource configuration
}

# outputs.tf - Output values
output "example_output" {
  description = "Description of the output"
  value       = aws_example.main.attribute
}
```

### 3. Environment File Requirements

Each environment folder (dev/staging/prod) MUST have:

#### main.tf
```hcl
##################################################################
# Main Configuration
# 
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
##################################################################

##################################################################
# Module Calls
# 
module "vpc" {
  source = "../.modules/vpc"
  
  # Resource naming variables
  name = local.common.project
  env  = local.common.env
  
  # Module-specific variables
  vpc_cidr        = local.vpc.cidr_block
  vpc_enable_dns  = local.vpc.enable_dns
  vpc_availability_zones = local.vpc.availability_zones
  
  # Common variables
  common_project_name = local.common.project
  common_environment  = local.common.env
  common_account      = local.common.account
}
##################################################################
```

#### locals.tf
```hcl
##################################################################
# Local Values
# 
locals {
  common = {
    project = "my-project-name"
    env     = "development"
    account = "123456789012"
  }

  vpc = {
    cidr_block        = "10.0.0.0/16"
    enable_dns        = true
    availability_zones = ["us-west-2a", "us-west-2b"]
  }

  ec2 = {
    instance_type  = "t3.micro"
    instance_count = 1
    ami_id         = "ami-12345678"
  }
}
##################################################################
```

#### providers.tf
```hcl
##################################################################
# Provider Configuration
# 
provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = local.common_tags
  }
}
##################################################################
```

#### versions.tf
```hcl
##################################################################
# Version Constraints
# 
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
##################################################################
```

#### state.tf
```hcl
##################################################################
# Backend Configuration
# 
terraform {
  backend "s3" {
    bucket         = "terraform-state-bucket"
    key            = "environments/${var.environment}/terraform.tfstate"
    region         = "us-west-2"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
##################################################################
```

## Comment Standards

### Module Separation Comments
Use this exact format for separating modules in Terraform files:

```hcl
##################################################################
# Module Name
# 
```

### Example Implementation
```hcl
##################################################################
# VPC Module
# 
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name        = "${var.name}-${var.env}-vpc"
    Environment = var.env
    Project     = var.name
    Account     = var.common_account
    ManagedBy   = "terraform"
  }
}
##################################################################

##################################################################
# Internet Gateway Module
# 
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name        = "${var.name}-${var.env}-igw"
    Environment = var.env
    Project     = var.name
    Account     = var.common_account
    ManagedBy   = "terraform"
  }
}
##################################################################

##################################################################
# Subnets Module
# 
resource "aws_subnet" "public" {
  count = length(var.availability_zones)

  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnet_cidrs[count.index]
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true

  tags = {
    Name        = "${var.name}-${var.env}-public-subnet-${count.index + 1}"
    Environment = var.env
    Project     = var.name
    Account     = var.common_account
    Type        = "public"
    ManagedBy   = "terraform"
  }
}
##################################################################
```

## Execution Scripts

### Required Bin Scripts
Each repository MUST have these execution scripts:

* bin/plan
* bin/apply

### Usage Examples
```bash
# Format and plan development environment
bin/plan dev

# Format and apply staging environment
bin/apply staging

# Format and plan production environment
bin/plan prod
```

## Validation Commands

Run these commands to validate your Terraform code:

```bash
# Use bin scripts
bin/plan dev
bin/apply dev
```

## File Structure Validation

Use this checklist for every repository:

- [ ] `.modules/` directory exists
- [ ] `dev/` directory exists with required files
- [ ] `staging/` directory exists with required files  
- [ ] `prod/` directory exists with required files
- [ ] `bin/` directory exists with `plan` and `apply` scripts
- [ ] Each module has `variables.tf`, `main.tf`, `outputs.tf`
- [ ] Each environment has `main.tf`, `locals.tf`, `providers.tf`, `versions.tf`, `state.tf`
- [ ] All `.tf` files use module separation comments
- [ ] Bin scripts are executable (`chmod +x bin/plan bin/apply`)
- [ ] Code passes `terraform fmt` and `terraform validate`
