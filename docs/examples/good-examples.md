# Good Examples - Terraform Code Standards

## Repository Structure Example

```
my-terraform-project/
├── modules/
│   ├── application-load-balancer/
│   │   ├── variables.tf
│   │   ├── main.tf
│   │   └── outputs.tf
│   ├── ecs/
│   │   ├── variables.tf
│   │   ├── main.tf
│   │   └── outputs.tf
│   └── dynamodb/
│       ├── variables.tf
│       ├── main.tf
│       └── outputs.tf
├── dev/
│   ├── main.tf
│   ├── locals.tf
│   ├── provider.tf
│   ├── versions.tf
│   ├── state.tf
│   ├── variables.tf
│   └── outputs.tf
├── staging/
│   ├── main.tf
│   ├── locals.tf
│   ├── provider.tf
│   ├── versions.tf
│   ├── state.tf
│   ├── variables.tf
│   └── outputs.tf
├── prod/
│   ├── main.tf
│   ├── locals.tf
│   ├── provider.tf
│   ├── versions.tf
│   ├── state.tf
│   ├── variables.tf
│   └── outputs.tf
├── bin/
│   ├── plan
│   └── apply
└── README.md
```

## Module Example - Application Load Balancer Module

### modules/application-load-balancer/variables.tf
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

variable "vpc_id" {
  description = "ID of the VPC where resources are hosted"
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
```

### modules/application-load-balancer/main.tf
```hcl
#############################################
# Application Load Balancer
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
# Target Group
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
# Security Group
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

#############################################
# Listener
#
resource "aws_alb_listener" "http" {
  load_balancer_arn = aws_lb.main.id
  port              = 80
  protocol          = "HTTP"

  default_action {
    target_group_arn = aws_alb_target_group.main.id
    type             = "forward"
  }

  lifecycle {
    prevent_destroy = true
  }
}
#############################################
```

### modules/application-load-balancer/outputs.tf
```hcl
#############################################
# Application Load Balancer Outputs
#
output "aws_alb_arn" {
  description = "ARN of the Application Load Balancer"
  value       = aws_lb.main.arn
}

output "aws_alb_dns" {
  description = "DNS name of the Application Load Balancer"
  value       = aws_lb.main.dns_name
}

output "aws_alb_target_group_arn" {
  description = "ARN of the target group"
  value       = aws_alb_target_group.main.arn
}

output "aws_alb_security_group_id" {
  description = "ID of the security group"
  value       = aws_security_group.main.id
}
#############################################
```

## Environment Example - Development Environment

### dev/main.tf
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

### dev/locals.tf
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

### dev/provider.tf
```hcl
provider "aws" {
  region = local.common.aws_region
}
```

### dev/versions.tf
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

### dev/state.tf
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
