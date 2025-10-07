# Terraform Infrastructure Coding Standards

## 🚨 **CRITICAL REQUIREMENTS FOR AI ASSISTANTS**

**MANDATORY PATTERNS TO ENFORCE:**
- Repository structure: `modules/`, `dev/`, `staging/`, `prod/`
- Module files: `variables.tf`, `main.tf`, `outputs.tf` (ALL REQUIRED)
- Environment files: `main.tf`, `locals.tf`, `provider.tf`, `versions.tf`, `state.tf` (ALL REQUIRED)
- README.md: Project info, requirements, and usage only (NO technical docs)
- Comment format: `#############################################` blocks with proper structure
- Resource naming: `"${var.name}-${var.env}-${resource_type}"`
- Variable naming: Module prefixes (e.g., `alb_`, `ecs_`, `dynamodb_`)
- Locals structure: `common` + module-specific sections
- Minimum tags: `Name`, `Environment`, `Project`

## Repository Structure Requirements

Every Terraform repository MUST follow this exact structure:

```
repository/
├── modules/           # Required: Custom Terraform modules
├── dev/               # Required: Development environment
├── staging/           # Required: Staging environment  
├── prod/              # Required: Production environment
└── README.md
```

## Module Structure Requirements

### Module Creation Rules
- **One module per service**: Create separate modules for each AWS service (cloudfront, route53, ecs, etc.)
- **Main resources only**: Each module should contain only the main resources for that service
- **Multiple resources**: If a service has multiple main resources, create separate `.tf` files with descriptive names

### Required Module Files
- `variables.tf` - Input variables for the module
- `main.tf` - Main resource definitions (ONLY if single main resource)
- `outputs.tf` - Output values from the module

### Module File Structure Example
```
modules/
├── cloudfront/
│   ├── variables.tf
│   ├── main.tf
│   └── outputs.tf
├── route53/
│   ├── variables.tf
│   ├── main.tf
│   └── outputs.tf
├── ecs/
│   ├── variables.tf
│   ├── main.tf
│   └── outputs.tf
└── dynamodb/
    ├── variables.tf
    ├── main.tf
    └── outputs.tf
```

### Multiple Resources Example
```
modules/
├── cloudfront/
│   ├── variables.tf
│   ├── distribution.tf
│   ├── origin-access-control.tf
│   └── outputs.tf
├── route53/
│   ├── variables.tf
│   ├── zone.tf
│   ├── record.tf
│   └── outputs.tf
```

## README.md Requirements

### Repository README.md Content
The main `README.md` file in the repository root MUST contain ONLY:

- **Project Information**: Brief description of what the project does
- **Requirements**: Prerequisites and dependencies needed
- **Usage Instructions**: How to plan and apply the infrastructure

### README.md Structure
```markdown
# Project Name

Brief description of the project and its purpose.

## Requirements

- Terraform >= 1.0.4
- AWS CLI configured
- Appropriate AWS permissions

## Usage

### Plan Infrastructure
```bash
bin/plan dev
bin/plan staging
bin/plan prod
```

### Apply Infrastructure
```bash
bin/apply dev
bin/apply staging
bin/apply prod
```
```

### README.md Rules
- **DO NOT include**: Technical documentation, standards, or detailed architecture
- **DO NOT include**: Module documentation or code examples
- **DO NOT include**: Development guidelines or coding standards
- **KEEP FOCUSED**: Only project info, requirements, and basic usage

## Environment Structure Requirements

Each environment folder (dev, staging, prod) MUST contain these files:

### Required Environment Files
- `main.tf` - Main configuration and module calls
- `locals.tf` - Local values and computed variables
- `provider.tf` - Provider configurations
- `versions.tf` - Terraform and provider version constraints
- `state.tf` - Backend configuration for state management

### Environment File Structure Example
```
dev/
├── main.tf
├── locals.tf
├── provider.tf
├── versions.tf
└── state.tf
```

## Terraform File Formatting Rules

### Module Comments Requirement
Every Terraform file MUST use `#` comments to separate different modules or logical sections.

### Example File Structure with Comments
```hcl
#############################################
# Common
# 
variable "name" {
  description = "Name of the project"
  type        = string
}

variable "env" {
  description = "Name of the environment"
  type        = string
}
#############################################

#############################################
# Application Load Balancer Module
# 
variable "alb_subnet_ids" {
  description = "Subnet IDS related to the application load balancer"
  type        = list(string)
}

variable "alb_health_check_path" {
  description = "Path to the health endpoint for the target group"
  type        = string
}
#############################################

#############################################
# ECS Module
# 
variable "ecs_task_cpu" {
  description = "ECS task CPU amount in milli vCPUs"
  type        = number
}

variable "ecs_task_memory" {
  description = "ECS task memory amount in MiB"
  type        = number
}
#############################################
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
#############################################
# Module Name
# 
resource "aws_resource" "name" {
  # Resource configuration
}
#############################################
```

### Example Implementation
```hcl
#############################################
# Application Load Balancer Module
# 
resource "aws_lb" "main" {
  name               = "${var.name}-${var.env}-lb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.main.id]
  subnets            = var.alb_subnet_ids

  enable_deletion_protection = false
  idle_timeout               = 600

  tags = {
    Name        = "${var.name}-${var.env}-lb"
    Environment = var.env
    Project     = var.name
  }
}
#############################################

#############################################
# Target Group Module
# 
resource "aws_alb_target_group" "main" {
  name     = "${var.name}-${var.env}-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = var.vpc_id

  health_check {
    enabled             = true
    healthy_threshold   = 2
    interval            = 30
    matcher             = "200"
    path                = var.alb_health_check_path
    port                = "traffic-port"
    protocol            = "HTTP"
    timeout             = 5
    unhealthy_threshold = 2
  }

  tags = {
    Name        = "${var.name}-${var.env}-tg"
    Environment = var.env
    Project     = var.name
  }
}
#############################################
```

## Locals Structure Requirements

### Required Locals Structure
All environment `locals.tf` files MUST follow this exact structure:

```hcl
locals {
  common = {
    name        = "project-name"
    environment = "dev"
    tags = {
      account = "dev"
    }
  }

  ecs = {
    ecs_field = "value"
    # ... other ecs-specific values
  }

  alb = {
    alb_field = "value"
    # ... other alb-specific values
  }
}
```

### Locals Structure Rules
- **common**: Contains shared values across all modules
- **Module-specific sections**: Each module gets its own section (ecs, alb, dynamodb, etc.)
- **No environment-specific variables**: All values are defined in locals, not variables
- **Consistent naming**: Use module prefixes for module-specific values

## Tagging Requirements

### Mandatory Tags
ALL resources that support tagging MUST include these minimum tags:

- `Name` - Resource name following naming convention
- `Environment` - Environment name
- `Project` - Project name

### Tagging Example
```hcl
tags = {
  Name        = "${var.name}-${var.env}-resource-type"
  Environment = var.env
  Project     = var.name
}
```

## Bin Scripts Requirements

### Bin Scripts
- **DO NOT CREATE**: `bin/plan` and `bin/apply` scripts are already provided
- **DO NOT MODIFY**: Existing bin scripts should not be altered
- **USAGE**: Use existing scripts for terraform operations

## Validation Checklist

Before committing any Terraform code, ensure:

- [ ] Repository has `modules/`, `dev/`, `staging/`, `prod/` folders
- [ ] README.md contains only project info, requirements, and usage instructions
- [ ] Each module has `variables.tf`, `main.tf`, `outputs.tf`
- [ ] Each environment has `main.tf`, `locals.tf`, `provider.tf`, `versions.tf`, `state.tf`
- [ ] All `.tf` files use `#############################################` comments to separate modules
- [ ] Resource names follow naming conventions
- [ ] All files pass `terraform fmt` and `terraform validate`
