# Anti-Patterns - What NOT to Do

## ❌ Bad Repository Structure

```
bad-terraform-project/
├── modules/                    # Wrong: should be .modules/
├── development/                # Wrong: should be dev/
├── production/                 # Wrong: should be prod/
├── test/                       # Wrong: should be staging/
└── main.tf                     # Wrong: files in root
```

## ❌ Bad Module Structure

```
.modules/
├── vpc/
│   ├── vpc.tf                  # Wrong: should be main.tf
│   ├── variables.tf
│   └── outputs.tf
├── ec2/
│   ├── main.tf
│   └── variables.tf            # Missing: outputs.tf
└── rds/
    └── main.tf                 # Missing: variables.tf and outputs.tf
```

## ❌ Bad Environment Structure

```
dev/
├── main.tf
├── variables.tf                # Wrong: should be locals.tf
├── outputs.tf                  # Wrong: should not be in environment
└── terraform.tfvars            # Wrong: should use locals.tf instead
```

## ❌ Bad Terraform File Formatting

```hcl
# Bad: No module separation comments
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
}

resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"
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
  default     = "us-west-2"
}
```

## ❌ Bad Resource Configuration

```hcl
# Bad: No tags, hardcoded values
resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t3.micro"
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
```

## ❌ Bad Module Usage

```hcl
# Bad: No source path
module "vpc" {
  # Missing source
  vpc_cidr = "10.0.0.0/16"
}

# Bad: Hardcoded values instead of variables
module "ec2" {
  source = "../.modules/ec2"
  
  instance_count = 2
  instance_type   = "t3.medium"
}

# Bad: No outputs usage
module "vpc" {
  source = "../.modules/vpc"
  
  vpc_cidr = var.vpc_cidr
}

# Then using hardcoded values instead of module outputs
resource "aws_subnet" "public" {
  vpc_id = "vpc-12345678"  # Should use module.vpc.vpc_id
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
  region = "us-west-2"
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
}

resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"
}

resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t3.micro"
  subnet_id     = aws_subnet.public.id
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
  region = "us-west-2"
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
