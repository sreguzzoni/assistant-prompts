# Anti-Patterns - What NOT to Do

## ❌ Bad Repository Structure

```
bad-terraform-project/
├── .modules/                   # Wrong: should be modules/
├── development/                # Wrong: should be dev/
├── production/                 # Wrong: should be prod/
├── test/                       # Wrong: should be staging/
└── main.tf                     # Wrong: files in root
```

## ❌ Bad Module Structure

```
modules/
├── application-load-balancer/
│   ├── alb.tf                  # Wrong: should be main.tf
│   ├── variables.tf
│   └── outputs.tf
├── ecs/
│   ├── main.tf
│   └── variables.tf            # Missing: outputs.tf
└── dynamodb/
    └── main.tf                 # Missing: variables.tf and outputs.tf
```

## ❌ Bad Environment Structure

```
dev/
├── main.tf
├── variables.tf                # Wrong: should be locals.tf
├── outputs.tf                  # Wrong: should not be in environment
├── providers.tf                # Wrong: should be provider.tf
└── terraform.tfvars            # Wrong: should use locals.tf instead
```

## ❌ Bad Terraform File Formatting

```hcl
# Bad: No module separation comments
resource "aws_lb" "main" {
  name               = "my-lb"
  load_balancer_type = "application"
  subnets            = var.subnet_ids
}

resource "aws_alb_target_group" "main" {
  name     = "my-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = var.vpc_id
}

resource "aws_security_group" "main" {
  name        = "my-sg"
  description = "Security group"
  vpc_id      = var.vpc_id
}

# Bad: Inconsistent naming
resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t3.micro"
}

# Bad: No tags
resource "aws_s3_bucket" "data" {
  bucket = "my-bucket"
}
```

## ❌ Bad Variable Definitions

```hcl
# Bad: No description, no type, no default
variable "name" {
}

# Bad: Vague description
variable "count" {
  description = "Number"
  type        = number
}

# Bad: Hardcoded values in variables
variable "region" {
  description = "AWS region"
  type        = string
  default     = "eu-west-1"
}

# Bad: No module prefix for module-specific variables
variable "subnet_ids" {
  description = "Subnet IDs"
  type        = list(string)
}
```

## ❌ Bad Resource Configuration

```hcl
# Bad: No tags, hardcoded values
resource "aws_lb" "main" {
  name               = "my-lb"
  load_balancer_type = "application"
  subnets            = ["subnet-123", "subnet-456"]
}

# Bad: Inconsistent naming
resource "aws_security_group" "sg" {
  name = "sg"
}

# Bad: No description in security group rules
resource "aws_security_group_rule" "allow_http" {
  type              = "ingress"
  from_port         = 80
  to_port           = 80
  protocol          = "tcp"
  cidr_blocks       = ["0.0.0.0/0"]
  security_group_id = aws_security_group.sg.id
}

# Bad: No health check configuration
resource "aws_alb_target_group" "main" {
  name     = "my-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = "vpc-123"
}
```

## ❌ Bad Module Usage

```hcl
# Bad: No source path
module "application_load_balancer" {
  # Missing source
  name = "my-project"
  env  = "dev"
}

# Bad: Hardcoded values instead of variables
module "ecs" {
  source = "../modules/ecs"
  
  name = "my-project"
  env  = "dev"
  ecs_task_cpu    = 512
  ecs_task_memory = 2048
}

# Bad: No outputs usage
module "application_load_balancer" {
  source = "../modules/application-load-balancer"
  
  name = var.name
  env  = var.env
}

# Then using hardcoded values instead of module outputs
resource "aws_alb_listener" "http" {
  load_balancer_arn = "arn:aws:elasticloadbalancing:..."  # Should use module.application_load_balancer.aws_alb_arn
}
```

## ❌ Bad File Organization

```hcl
# Bad: Everything in one file
# main.tf - Contains everything
terraform {
  required_version = ">= 1.0"
}

provider "aws" {
  region = "eu-west-1"
}

resource "aws_lb" "main" {
  name               = "my-lb"
  load_balancer_type = "application"
  subnets            = ["subnet-123", "subnet-456"]
}

resource "aws_alb_target_group" "main" {
  name     = "my-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = "vpc-123"
}

resource "aws_security_group" "main" {
  name        = "my-sg"
  description = "Security group"
  vpc_id      = "vpc-123"
}

resource "aws_alb_listener" "http" {
  load_balancer_arn = aws_lb.main.arn
  port              = 80
  protocol          = "HTTP"
  default_action {
    target_group_arn = aws_alb_target_group.main.arn
    type             = "forward"
  }
}

# Bad: No separation of concerns
```

## ❌ Bad State Management

```hcl
# Bad: No backend configuration
terraform {
  required_version = ">= 1.0"
}

# Bad: Local state in production
# No state.tf file

# Bad: Insecure backend
terraform {
  backend "s3" {
    bucket = "terraform-state"
    key    = "terraform.tfstate"
    # Missing: region, encrypt, dynamodb_table
  }
}

# Bad: Hardcoded backend values
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

## ❌ Bad Version Management

```hcl
# Bad: No version constraints
terraform {
  # Missing required_version
}

# Bad: Too restrictive versions
terraform {
  required_version = "= 1.0.0"  # Should be >= 1.0
}

# Bad: No provider version constraints
provider "aws" {
  region = "eu-west-1"
}

# Bad: Too specific provider versions
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "= 5.95.0"  # Should be ~> 5.95.0
    }
  }
}
```

## Common Mistakes to Avoid

1. **Missing Required Files**: Not having all required files in modules/environments
2. **Inconsistent Naming**: Using different naming conventions across resources
3. **No Comments**: Not using module separation comments
4. **Hardcoded Values**: Using hardcoded values instead of variables
5. **Missing Tags**: Not tagging resources properly
6. **Poor Structure**: Not following the required directory structure
7. **No State Management**: Not configuring secure backend for state
8. **Version Issues**: Not specifying version constraints
9. **Security Issues**: Using overly permissive security groups
10. **No Validation**: Not running `terraform fmt` and `terraform validate`
