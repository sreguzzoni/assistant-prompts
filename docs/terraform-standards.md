# Terraform Standards and Best Practices

## File Organization Standards

### 1. Repository Structure Enforcement

**MANDATORY**: Every repository must have this exact structure:

```
terraform-repository/
├── modules/          # Custom modules directory
├── dev/              # Development environment
├── staging/          # Staging environment
├── prod/             # Production environment
├── bin/              # Execution scripts directory
│   ├── plan          # Terraform plan script
│   └── apply         # Terraform apply script
└── README.md
```

### 2. Module File Requirements

Each module in `modules/` directory MUST have:

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
module "modules" {
  source = "../modules"

  name                = local.common.name
  env                 = local.common.env
  aws_root_account_id = local.common.aws_root_account_id
  aws_region          = local.common.aws_region
  vpc_id              = local.common.vpc_id
  igw_id              = local.common.igw_id
  nat_id              = local.common.nat_id

  alb_health_check_path = local.application_load_balancer.alb_health_check_path

  cloudfront_aliases             = local.cloudfront.cloudfront_aliases
  cloudfront_enable_acm_creation = local.cloudfront.cloudfront_enable_acm_creation
  cloudfront_certificate_arn     = local.cloudfront.cloudfront_certificate_arn
  cloudfront_hosted_zone         = local.cloudfront.cloudfront_hosted_zone

  ecr_repository_lifecycle_count_threshold = local.ecr.ecr_repository_lifecycle_count_threshold

  ecs_task_container_definitions = local.ecs.ecs_task_container_definitions
  ecs_task_cpu                   = local.ecs.ecs_task_cpu
  ecs_task_memory                = local.ecs.ecs_task_memory
  ecs_desired_capacity           = local.ecs.ecs_desired_capacity
  ecs_max_capacity               = local.ecs.ecs_max_capacity
  ecs_min_capacity               = local.ecs.ecs_min_capacity
  ecs_task_log_group_region      = local.ecs.ecs_task_log_group_region
  ecs_task_log_group_name        = local.ecs.ecs_task_log_group_name

  iam_gitlab_ci_root_role = local.iam.iam_gitlab_ci_root_role

  rt_vpc_s3_endpoint_id = local.route_table.rt_vpc_s3_endpoint_id

  subnet_availability_zones = local.subnet.subnet_availability_zones
  subnet_public_subnets     = local.subnet.subnet_public_subnets
  subnet_private_subnets    = local.subnet.subnet_private_subnets

  dynamodb_deletion_protection_enabled   = local.dynamodb.dynamodb_deletion_protection_enabled
  dynamodb_enable_point_in_time_recovery = local.dynamodb.dynamodb_enable_point_in_time_recovery
}
```

#### locals.tf
```hcl
locals {
  common = {
    name                = "my-project-name"
    env                 = "dev"
    aws_root_account_id = "564989531723"
    aws_account_id      = "990845929947"
    aws_region          = "eu-west-1"
    vpc_id              = "vpc-006c63b1626b611cf"
    igw_id              = "igw-0bb50c8e681523ae2"
    nat_id              = "nat-0641602821bcc70cc"
  }

  application_load_balancer = {
    alb_health_check_path = "/health"
  }

  cloudfront = {
    cloudfront_aliases = [
      "my-project.${local.common.env}.example.com",
    ]
    cloudfront_enable_acm_creation = true
    cloudfront_certificate_arn     = ""
    cloudfront_hosted_zone         = "my-project.${local.common.env}.example.com"
  }

  dynamodb = {
    dynamodb_deletion_protection_enabled   = true
    dynamodb_enable_point_in_time_recovery = false
  }

  ecr = {
    ecr_repository_lifecycle_count_threshold = 10
  }

  ecs = {
    ecs_task_container_definitions = jsonencode([
      {
        name        = "${local.common.name}-${local.common.env}-application"
        image       = "${local.common.aws_account_id}.dkr.ecr.eu-west-1.amazonaws.com/${local.common.name}-${local.common.env}-application-repository:latest"
        essential   = true
        networkMode = "awsvpc"
        logConfiguration = {
          logDriver = "awslogs"
          options = {
            awslogs-create-group  = "True"
            awslogs-group         = "${local.common.name}/${local.common.env}"
            awslogs-region        = local.common.aws_region
            awslogs-stream-prefix = "application"
          }
        }
      }
    ])
    ecs_task_cpu              = 512
    ecs_task_memory           = 2048
    ecs_desired_capacity      = 1
    ecs_max_capacity          = 1
    ecs_min_capacity          = 1
    ecs_task_log_group_region = local.common.aws_region
    ecs_task_log_group_name   = "${local.common.name}/${local.common.env}"
  }

  iam = {
    iam_gitlab_ci_root_role = "gitlab-ci"
  }

  route_table = {
    rt_vpc_s3_endpoint_id = "vpce-0c46b1c93f61f445b"
  }

  subnet = {
    subnet_availability_zones = [
      "eu-west-1a",
      "eu-west-1b",
    ]
    subnet_public_subnets = [
      "10.10.132.0/27",
      "10.10.132.32/27",
    ]
    subnet_private_subnets = [
      "10.10.132.128/27",
      "10.10.132.160/27",
    ]
  }
}
```

#### provider.tf
```hcl
provider "aws" {
  region = local.common.aws_region
}
```

#### versions.tf
```hcl
terraform {
  required_version = ">= 1.0.4"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "5.95.0"
    }
  }
}
```

#### state.tf
```hcl
terraform {
  backend "s3" {
    bucket         = "infra.dev.example"
    key            = "terraform/dev/my-project/terraform.tfstate"
    region         = "eu-west-1"
    dynamodb_table = "terraform_locks"
    encrypt        = true
  }
}
```

## Comment Standards

### Module Separation Comments
Use this exact format for separating modules in Terraform files:

```hcl
#############################################
# Module Name
# 
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

#############################################
# Security Group Module
# 
resource "aws_security_group" "main" {
  name        = "${var.name}-${var.env}-sg"
  description = "Security group for application load balancer"
  vpc_id      = var.vpc_id

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

  tags = {
    Name        = "${var.name}-${var.env}-sg"
    Environment = var.env
    Project     = var.name
  }
}
#############################################
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

- [ ] `modules/` directory exists
- [ ] `dev/` directory exists with required files
- [ ] `staging/` directory exists with required files  
- [ ] `prod/` directory exists with required files
- [ ] `bin/` directory exists with `plan` and `apply` scripts
- [ ] Each module has `variables.tf`, `main.tf`, `outputs.tf`
- [ ] Each environment has `main.tf`, `locals.tf`, `provider.tf`, `versions.tf`, `state.tf`
- [ ] All `.tf` files use module separation comments
- [ ] Bin scripts are executable (`chmod +x bin/plan bin/apply`)
- [ ] Code passes `terraform fmt` and `terraform validate`
