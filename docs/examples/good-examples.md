# Good Examples - Terraform Code Standards

## Repository Structure Example

```
my-terraform-project/
├── .modules/
│   ├── vpc/
│   │   ├── variables.tf
│   │   ├── main.tf
│   │   └── outputs.tf
│   ├── ec2/
│   │   ├── variables.tf
│   │   ├── main.tf
│   │   └── outputs.tf
│   └── rds/
│       ├── variables.tf
│       ├── main.tf
│       └── outputs.tf
├── dev/
│   ├── main.tf
│   ├── locals.tf
│   ├── providers.tf
│   ├── versions.tf
│   └── state.tf
├── staging/
│   ├── main.tf
│   ├── locals.tf
│   ├── providers.tf
│   ├── versions.tf
│   └── state.tf
├── prod/
│   ├── main.tf
│   ├── locals.tf
│   ├── providers.tf
│   ├── versions.tf
│   └── state.tf
├── bin/
│   ├── plan
│   └── apply
└── README.md
```

## Module Example - VPC Module

### .modules/vpc/variables.tf
```hcl
##################################################################
# VPC Module Variables
#
variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "vpc_enable_dns" {
  description = "Enable DNS support in VPC"
  type        = bool
  default     = true
}

variable "vpc_availability_zones" {
  description = "List of availability zones"
  type        = list(string)
  default     = ["us-west-2a", "us-west-2b"]
}

# Resource naming variables
variable "name" {
  description = "Base name for resources"
  type        = string
}

variable "env" {
  description = "Environment name"
  type        = string
}

# Common variables (no prefix)
variable "common_project_name" {
  description = "Name of the project"
  type        = string
}

variable "common_environment" {
  description = "Environment name"
  type        = string
}

variable "common_account" {
  description = "Account identifier"
  type        = string
}
##################################################################
```

### .modules/vpc/main.tf
```hcl
##################################################################
# VPC Module Main Configuration
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
# Public Subnets Module
#
resource "aws_subnet" "public" {
  count = length(var.availability_zones)

  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 8, count.index)
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

##################################################################
# Private Subnets Module
#
resource "aws_subnet" "private" {
  count = length(var.availability_zones)

  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(var.vpc_cidr, 8, count.index + 10)
  availability_zone = var.availability_zones[count.index]

  tags = {
    Name        = "${var.name}-${var.env}-private-subnet-${count.index + 1}"
    Environment = var.env
    Project     = var.name
    Account     = var.common_account
    Type        = "private"
    ManagedBy   = "terraform"
  }
}
##################################################################
```

### .modules/vpc/outputs.tf
```hcl
##################################################################
# VPC Module Outputs
#
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}

output "vpc_cidr_block" {
  description = "CIDR block of the VPC"
  value       = aws_vpc.main.cidr_block
}

output "public_subnet_ids" {
  description = "IDs of the public subnets"
  value       = aws_subnet.public[*].id
}

output "private_subnet_ids" {
  description = "IDs of the private subnets"
  value       = aws_subnet.private[*].id
}

output "internet_gateway_id" {
  description = "ID of the Internet Gateway"
  value       = aws_internet_gateway.main.id
}
##################################################################
```

## Environment Example - Development Environment

### dev/main.tf
```hcl
##################################################################
# VPC Module Call
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

##################################################################
# EC2 Module Call
# 
module "web_servers" {
  source = "../.modules/ec2"
  
  # Resource naming variables
  name = local.common.project
  env  = local.common.env
  
  # Module-specific variables
  ec2_instance_count = local.ec2.instance_count
  ec2_instance_type  = local.ec2.instance_type
  ec2_ami_id         = local.ec2.ami_id
  ec2_subnet_ids     = module.vpc.public_subnet_ids
  ec2_vpc_id         = module.vpc.vpc_id
  
  # Common variables
  common_project_name = local.common.project
  common_environment  = local.common.env
  common_account      = local.common.account
}
##################################################################
```

### dev/locals.tf
```hcl
##################################################################
# Development Environment Local Values
# 
locals {
  common = {
    project = "my-terraform-project"
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

  rds = {
    instance_class    = "db.t3.micro"
    allocated_storage = 20
    engine           = "mysql"
  }
}
##################################################################
```

### dev/providers.tf
```hcl
##################################################################
# Provider Configuration for Development
# 
provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = local.common_tags
  }
}
##################################################################
```

### dev/versions.tf
```hcl
##################################################################
# Version Constraints for Development
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

### dev/state.tf
```hcl
##################################################################
# Backend Configuration for Development
# 
terraform {
  backend "s3" {
    bucket         = "my-company-terraform-state"
    key            = "environments/development/terraform.tfstate"
    region         = "us-west-2"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
##################################################################
```

## Key Best Practices Demonstrated

1. **Consistent Structure**: All modules follow the same file structure
2. **Clear Comments**: Module separation with standardized comment format
3. **Descriptive Naming**: Resources and variables use clear, descriptive names
4. **Tagging Strategy**: Consistent tagging across all resources
5. **Modular Design**: Reusable modules with clear inputs and outputs
6. **Environment Separation**: Clear separation between environments
7. **Version Management**: Proper version constraints for Terraform and providers
8. **State Management**: Secure backend configuration for state storage
9. **Execution Scripts**: Standardized bin scripts for consistent operations
