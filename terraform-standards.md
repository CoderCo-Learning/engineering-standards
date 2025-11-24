# Terraform Standards

## Project Structure

### Standard Module Layout

```
terraform-aws-vpc/
├── README.md
├── main.tf
├── variables.tf
├── outputs.tf
├── versions.tf
├── examples/
│   ├── basic/
│   │   ├── main.tf
│   │   └── README.md
│   └── complete/
│       ├── main.tf
│       └── README.md
├── modules/
│   └── subnets/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
└── tests/
    └── integration_test.go
```

### File Organisation

#### versions.tf

```hcl
terraform {
  required_version = ">= 1.5.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = ">= 5.0"
    }
  }
}
```

#### variables.tf
All input variables with descriptions

#### outputs.tf
All outputs with descriptions

#### main.tf
Main resource definitions

#### data.tf (optional)
Data sources only

#### locals.tf (optional)
Local values only

## Naming Conventions

### Resources
```hcl
# Format: <resource_type>_<descriptive_name>
resource "aws_vpc" "main" { }
resource "aws_subnet" "private" { }
resource "aws_security_group" "web_server" { }
```

### Variables
```hcl
# Use snake_case
variable "vpc_cidr_block" { }
variable "enable_dns_hostnames" { }
variable "instance_type" { }
```

### Outputs
```hcl
# Use snake_case, descriptive names
output "vpc_id" { }
output "private_subnet_ids" { }
output "security_group_id" { }
```

### Modules
```hcl
# Use descriptive names
module "vpc" { }
module "application_load_balancer" { }
module "rds_cluster" { }
```

## Variable Standards

### Always Include
1. Description
2. Type
3. Default (when appropriate)
4. Validation (when needed)

### Good Variable Example
```hcl
variable "instance_type" {
  description = "EC2 instance type for application servers"
  type        = string
  default     = "t3.micro"
  
  validation {
    condition     = can(regex("^t3\\.", var.instance_type))
    error_message = "Instance type must be from t3 family."
  }
}

variable "environment" {
  description = "Environment name (e.g., dev, staging, prod)"
  type        = string
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "tags" {
  description = "Additional tags to apply to all resources"
  type        = map(string)
  default     = {}
}
```

### Variable Types

#### Simple Types
```hcl
variable "string_var" {
  type    = string
  default = "value"
}

variable "number_var" {
  type    = number
  default = 42
}

variable "bool_var" {
  type    = bool
  default = true
}
```

#### Complex Types
```hcl
variable "list_var" {
  type    = list(string)
  default = ["item1", "item2"]
}

variable "map_var" {
  type = map(string)
  default = {
    key1 = "value1"
    key2 = "value2"
  }
}

variable "object_var" {
  type = object({
    name   = string
    port   = number
    health_check = object({
      path     = string
      interval = number
    })
  })
}
```

### Input Validation

Always validate inputs that have constraints:

```hcl
variable "cidr_block" {
  description = "CIDR block for VPC"
  type        = string
  
  validation {
    condition     = can(cidrhost(var.cidr_block, 0))
    error_message = "Must be valid IPv4 CIDR block."
  }
}

variable "port" {
  description = "Port number for application"
  type        = number
  
  validation {
    condition     = var.port > 0 && var.port < 65536
    error_message = "Port must be between 1 and 65535."
  }
}
```

## Output Standards

### Always Include Description
```hcl
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}

output "private_subnet_ids" {
  description = "List of IDs of private subnets"
  value       = aws_subnet.private[*].id
}
```

### Mark Sensitive Outputs
```hcl
output "database_password" {
  description = "Master password for database"
  value       = random_password.db.result
  sensitive   = true
}
```

### Output Useful Information
```hcl
output "connection_info" {
  description = "Connection information for the application"
  value = {
    endpoint = aws_lb.main.dns_name
    port     = 443
    url      = "https://${aws_lb.main.dns_name}"
  }
}
```

## Resource Standards

### Use Descriptive Names
```hcl
# ✅ Good
resource "aws_security_group" "web_server" { }
resource "aws_iam_role" "ecs_task_execution" { }

# ❌ Bad
resource "aws_security_group" "sg1" { }
resource "aws_iam_role" "role" { }
```

### Always Add Tags
```hcl
resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr_block
  
  tags = merge(
    var.tags,
    {
      Name        = "${var.project_name}-vpc"
      Environment = var.environment
      ManagedBy   = "Terraform"
    }
  )
}
```

### Use Dynamic Blocks Wisely
```hcl
# Good use case - variable number of items
resource "aws_security_group" "main" {
  name   = "example"
  vpc_id = aws_vpc.main.id
  
  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = "tcp"
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}
```

### Avoid Hardcoded Values
```hcl
# ❌ Bad
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  subnet_id     = "subnet-12345"
}

# ✅ Good
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  subnet_id     = var.subnet_id
}
```

## Module Standards

### README.md Template
```markdown
# Module Name

Brief description of what this module does.

## Usage

```hcl
module "example" {
  source = "./modules/example"
  
  name        = "my-example"
  environment = "prod"
}
```

## Requirements

| Name | Version |
|------|---------|
| terraform | >= 1.5 |
| aws | >= 5.0 |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| name | Resource name | string | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| id | Resource ID |

## Examples

See [examples/](./examples/) directory.
```

### Module Versioning
```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.1.0"  # Pin to specific version
  
  # ... configuration
}
```

### Module Inputs Best Practices
```hcl
# Provide sensible defaults
variable "enable_nat_gateway" {
  description = "Enable NAT gateway for private subnets"
  type        = bool
  default     = true
}

# Make required things obvious
variable "vpc_id" {
  description = "ID of the VPC (required)"
  type        = string
  # No default = required
}
```

## State Management

### Remote State
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/vpc/terraform.tfstate"
    region         = "eu-west-2"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

### State Best Practices
- Always use remote state for team projects
- Enable state locking
- Encrypt state at rest
- Use separate states for different environments
- Never commit state files to git

## Security Best Practices

### Never Hardcode Secrets
```hcl
# ❌ Bad - secret in code
resource "aws_db_instance" "main" {
  password = "SuperSecret123!"
}

# ✅ Good - use secrets manager or variables
resource "aws_db_instance" "main" {
  password = random_password.db.result
}

resource "random_password" "db" {
  length  = 32
  special = true
}
```

### Use Data Sources for Sensitive Info
```hcl
data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "prod/db/password"
}

resource "aws_db_instance" "main" {
  password = data.aws_secretsmanager_secret_version.db_password.secret_string
}
```

### Restrict Resource Access
```hcl
resource "aws_s3_bucket" "main" {
  bucket = "my-bucket"
}

resource "aws_s3_bucket_public_access_block" "main" {
  bucket = aws_s3_bucket.main.id
  
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

## Testing

### Always Provide Examples
```
examples/
├── basic/
│   ├── main.tf
│   ├── outputs.tf
│   └── README.md
├── complete/
│   ├── main.tf
│   ├── outputs.tf
│   └── README.md
```

### Use terraform validate
```bash
terraform validate
```

### Use terraform fmt
```bash
terraform fmt -recursive
```

### Consider Automated Tests
```go
// tests/integration_test.go
func TestTerraformVPC(t *testing.T) {
    terraformOptions := &terraform.Options{
        TerraformDir: "../examples/basic",
    }
    
    defer terraform.Destroy(t, terraformOptions)
    terraform.InitAndApply(t, terraformOptions)
    
    vpcID := terraform.Output(t, terraformOptions, "vpc_id")
    assert.NotEmpty(t, vpcID)
}
```

## Code Style

### Formatting
- Use `terraform fmt` before committing
- 2 spaces for indentation
- Blank line between resource blocks

### Ordering
1. `terraform` block
2. `provider` blocks
3. `data` sources
4. `locals`
5. `resource` blocks
6. `module` blocks

### Comments
```hcl
# Comment for humans
resource "aws_vpc" "main" {
  # This CIDR block provides 65,536 IPs
  cidr_block = "10.0.0.0/16"
  
  # Enable DNS for Route53 private zones
  enable_dns_support   = true
  enable_dns_hostnames = true
}
```

## Common Patterns

### Count vs For_Each

#### Use count for simple replication
```hcl
resource "aws_subnet" "private" {
  count             = 3
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(var.vpc_cidr, 8, count.index)
  availability_zone = data.aws_availability_zones.available.names[count.index]
}
```

#### Use for_each for maps/sets
```hcl
resource "aws_subnet" "private" {
  for_each = var.private_subnets
  
  vpc_id            = aws_vpc.main.id
  cidr_block        = each.value.cidr
  availability_zone = each.value.az
  
  tags = {
    Name = each.key
  }
}
```

### Conditional Resources
```hcl
resource "aws_nat_gateway" "main" {
  count = var.enable_nat_gateway ? 1 : 0
  
  allocation_id = aws_eip.nat[0].id
  subnet_id     = aws_subnet.public[0].id
}
```

### Locals for Computed Values
```hcl
locals {
  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
  
  az_count = min(length(data.aws_availability_zones.available.names), 3)
}

resource "aws_subnet" "private" {
  count = local.az_count
  # ...
  tags = merge(local.common_tags, { Name = "private-${count.index}" })
}
```

## Common Mistakes to Avoid

### ❌ Don't
- Hardcode values that should be variables
- Store secrets in code
- Use generic resource names
- Skip validation on inputs
- Forget to add descriptions
- Mix environments in same state
- Commit `.terraform/` directory
- Use `terraform apply -auto-approve` in CI without review

### ✅ Do
- Use variables for all configurable values
- Use secrets manager for sensitive data
- Use descriptive, consistent naming
- Validate all user inputs
- Document everything
- Separate state per environment
- Use `.gitignore` properly
- Require approval for production changes

## .gitignore for Terraform

```gitignore
# Local .terraform directories
**/.terraform/*

# .tfstate files
*.tfstate
*.tfstate.*

# Crash log files
crash.log
crash.*.log

# Exclude all .tfvars files
*.tfvars
*.tfvars.json

# Ignore override files
override.tf
override.tf.json
*_override.tf
*_override.tf.json

# Include override files you wish to add
# !example_override.tf

# Include tfplan files
*tfplan*

# Ignore CLI configuration files
.terraformrc
terraform.rc
```

## Pre-commit Hooks

Example `.pre-commit-config.yaml`:
```yaml
repos:
  - repo: https://github.com/antonbabenko/pre-commit-terraform
    rev: v1.83.5
    hooks:
      - id: terraform_fmt
      - id: terraform_validate
      - id: terraform_docs
      - id: terraform_tflint
```

## Quick Reference

### Essential Commands
```bash

# init
terraform init

# Format
terraform fmt -recursive

# Validate
terraform validate

# Plan
terraform plan -out=tfplan

# Apply
terraform apply tfplan

# Destroy
terraform destroy
```

### Best Practices Checklist
- [ ] Used descriptive resource names
- [ ] Added descriptions to all variables and outputs
- [ ] Validated all inputs
- [ ] Tagged all resources
- [ ] Pinned provider versions
- [ ] Used remote state
- [ ] No hardcoded values
- [ ] No secrets in code
- [ ] Ran terraform fmt
- [ ] Provided examples
- [ ] Updated README

---
