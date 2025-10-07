# Terraform Infrastructure Coding Standards

## Repository Structure Requirements

Every Terraform repository MUST follow this exact structure:

```
repository/
├── .modules/           # Required: Custom Terraform modules
├── dev/               # Required: Development environment
├── staging/           # Required: Staging environment  
├── prod/              # Required: Production environment
├── bin/               # Required: Execution scripts
│   ├── plan           # Required: Terraform plan script
│   └── apply          # Required: Terraform apply script
└── README.md
```

## Module Structure Requirements

Each Terraform module folder MUST contain these three files:

### Required Module Files
- `variables.tf` - Input variables for the module
- `main.tf` - Main resource definitions
- `outputs.tf` - Output values from the module

### Module File Structure Example
```
.modules/
├── vpc/
│   ├── variables.tf
│   ├── main.tf
│   └── outputs.tf
├── ec2/
│   ├── variables.tf
│   ├── main.tf
│   └── outputs.tf
└── rds/
    ├── variables.tf
    ├── main.tf
    └── outputs.tf
```

## Environment Structure Requirements

Each environment folder (dev, staging, prod) MUST contain these five files:

### Required Environment Files
- `main.tf` - Main configuration and module calls
- `locals.tf` - Local values and computed variables
- `providers.tf` - Provider configurations
- `versions.tf` - Terraform and provider version constraints
- `state.tf` - Backend configuration for state management

### Environment File Structure Example
```
dev/
├── main.tf
├── locals.tf
├── providers.tf
├── versions.tf
└── state.tf
```

## Terraform File Formatting Rules

### Module Comments Requirement
Every Terraform file MUST use `#` comments to separate different modules or logical sections.

### Example File Structure with Comments
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

## File Naming Conventions

### Module Files
- Use descriptive names that indicate the module's purpose
- Use kebab case for all file names
- Use snake case for all the resources

## Resource Naming Convention

### Required Resource Naming Pattern
ALL resources MUST follow this exact naming pattern:

```
"${var.name}-${var.env}-${resource_type}"
```

### Examples
```hcl
# VPC Resource
resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr
  
  tags = {
    Name = "${var.name}-${var.env}-vpc"
    # ... other tags
  }
}

# EC2 Instance
resource "aws_instance" "web" {
  ami           = var.ec2_ami_id
  instance_type = var.ec2_instance_type
  
  tags = {
    Name = "${var.name}-${var.env}-ec2"
    # ... other tags
  }
}

# S3 Bucket
resource "aws_s3_bucket" "data" {
  bucket = "${var.name}-${var.env}-data-bucket"
  
  tags = {
    Name = "${var.name}-${var.env}-s3"
    # ... other tags
  }
}
```

### Variable Requirements
To support this naming convention, each module MUST have these variables:

```hcl
# In module variables.tf
variable "name" {
  description = "Base name for resources"
  type        = string
}

variable "env" {
  description = "Environment name"
  type        = string
}
```

## Variable Naming Conventions

### Module-Specific Variables
Variables related to specific modules MUST start with a prefix equal to the module name:

```hcl
# In .modules/vpc/variables.tf
variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
}

variable "vpc_enable_dns" {
  description = "Enable DNS support in VPC"
  type        = bool
  default     = true
}

# In .modules/ec2/variables.tf
variable "ec2_instance_type" {
  description = "EC2 instance type"
  type        = string
}

variable "ec2_ami_id" {
  description = "AMI ID for EC2 instances"
  type        = string
}
```

### Common Variables
Variables used across different modules are named "common" and have NO prefix:

```hcl
# In environment folders
variable "common_project_name" {
  description = "Name of the project"
  type        = string
}

variable "common_environment" {
  description = "Environment name"
  type        = string
}

variable "common_aws_region" {
  description = "AWS region"
  type        = string
}
```

## Comment Format Standards

### Required Comment Format
All Terraform files MUST use this exact comment format for module separation:

```hcl
##################################################################
# Module Name
# 
resource "aws_resource" "name" {
  # Resource configuration
}
##################################################################
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
```

## Tagging Requirements

### Mandatory Tags
ALL resources that support tagging MUST include these minimum tags:

- `project` - Project name
- `account` - Account identifier
- `env` - Environment name

### Tagging Example
```hcl
tags = {
  Name        = "${var.name}-${var.env}-resource-type"
  Environment = var.env
  Project     = var.name
  Account     = var.common_account
  ManagedBy   = "terraform"
}
```

## Bin Scripts Requirements

### Required Bin Files
Each repository MUST have these two bin files:

- bin/plan
- bin/apply

### Usage Examples
```bash
# Format and plan development environment
bin/plan dev

# Format and apply staging environment
bin/apply staging

# Format and plan production environment
bin/plan prod
```

## Validation Checklist

Before committing any Terraform code, ensure:

- [ ] Repository has `.modules/`, `dev/`, `staging/`, `prod/` folders
- [ ] Repository has `bin/plan` and `bin/apply` scripts
- [ ] Each module has `variables.tf`, `main.tf`, `outputs.tf`
- [ ] Each environment has `main.tf`, `locals.tf`, `providers.tf`, `versions.tf`, `state.tf`
- [ ] All `.tf` files use `##################################################################` comments to separate modules
- [ ] Resource names follow naming conventions
- [ ] All files pass `terraform fmt` and `terraform validate`
