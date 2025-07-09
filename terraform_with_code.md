# Terraform Interview Guide - 6 Years Experience

## Core Terraform Concepts

### Q1: Explain Terraform's execution workflow and state management in detail.

**Answer:**
Terraform follows a declarative approach with these phases:
1. **Init**: Downloads providers, modules, and initializes backend
2. **Plan**: Creates execution plan by comparing current state with desired state
3. **Apply**: Executes the plan to reach desired state
4. **Destroy**: Removes all managed resources

State management is crucial:
- **Local State**: Stored in `terraform.tfstate` file locally
- **Remote State**: Stored in backends like S3, Azure Storage, etc.
- **State Locking**: Prevents concurrent operations using DynamoDB, Consul, etc.
- **State Versioning**: Keeps historical versions for rollback capabilities

**Common Failure Scenario:**
State file corruption or inconsistency between actual infrastructure and state.

**Solution:**
```bash
# Import existing resources
terraform import aws_instance.example i-1234567890abcdef0

# Refresh state to sync with actual infrastructure
terraform refresh

# Force unlock if state is locked
terraform force-unlock LOCK_ID
```

### Q2: How do you handle sensitive data in Terraform, and what are the security best practices?

**Answer:**
**Sensitive Data Handling:**
1. **Sensitive Variables**: Mark variables as sensitive
```hcl
variable "db_password" {
  description = "Database password"
  type        = string
  sensitive   = true
}
```

2. **External Secret Management**:
```hcl
data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "prod/db/password"
}

resource "aws_db_instance" "main" {
  password = data.aws_secretsmanager_secret_version.db_password.secret_string
}
```

**Security Best Practices:**
- Use remote state with encryption
- Implement least privilege IAM policies
- Use service accounts/roles instead of access keys
- Enable state versioning and backup
- Use .gitignore for sensitive files

**Problem & Solution:**
**Problem**: Secrets exposed in state file
**Solution**: Use data sources for secrets and mark outputs as sensitive:
```hcl
output "connection_string" {
  value     = "server=${aws_db_instance.main.endpoint};password=${random_password.db.result}"
  sensitive = true
}
```

## Advanced State Management

### Q3: Explain state drift and how you handle it in production environments.

**Answer:**
**State Drift** occurs when actual infrastructure differs from Terraform state due to:
- Manual changes outside Terraform
- Failed applies
- External automation tools

**Detection Methods:**
```bash
# Detect drift
terraform plan -detailed-exitcode

# Refresh state
terraform refresh

# Show current state
terraform show
```

**Handling Strategies:**
1. **Prevention**: Use SCM policies, RBAC, monitoring
2. **Detection**: Regular plan checks, drift detection automation
3. **Remediation**: 
   - Import resources: `terraform import`
   - Remove from state: `terraform state rm`
   - Force replacement: `terraform apply -replace`

**Production Example:**
```bash
# Automated drift detection script
#!/bin/bash
terraform plan -detailed-exitcode
EXIT_CODE=$?
if [ $EXIT_CODE -eq 2 ]; then
    echo "Drift detected, sending alert..."
    # Send notification
fi
```

### Q4: How do you perform zero-downtime deployments using Terraform?

**Answer:**
**Strategies:**

1. **Blue-Green Deployment:**
```hcl
resource "aws_launch_template" "app" {
  name_prefix   = "app-${var.environment}-"
  image_id      = var.ami_id
  instance_type = var.instance_type

  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_autoscaling_group" "app" {
  name                = "app-${var.environment}-${aws_launch_template.app.latest_version}"
  vpc_zone_identifier = var.subnet_ids
  target_group_arns   = [aws_lb_target_group.app.arn]
  
  min_size         = var.min_capacity
  max_size         = var.max_capacity
  desired_capacity = var.desired_capacity

  lifecycle {
    create_before_destroy = true
  }
}
```

2. **Rolling Updates with Health Checks:**
```hcl
resource "aws_autoscaling_group" "app" {
  health_check_type         = "ELB"
  health_check_grace_period = 300
  wait_for_capacity_timeout = "10m"
  
  instance_refresh {
    strategy = "Rolling"
    preferences {
      min_healthy_percentage = 50
      instance_warmup       = 300
    }
  }
}
```

**Failure Scenario:**
Deployment fails halfway through, leaving infrastructure in inconsistent state.

**Solution:**
```bash
# Implement proper error handling
terraform apply -auto-approve
if [ $? -ne 0 ]; then
    echo "Apply failed, initiating rollback..."
    terraform apply -auto-approve -var="ami_id=${PREVIOUS_AMI}"
fi
```

## Modules and Code Organization

### Q5: Design a scalable module structure for a multi-environment, multi-region setup.

**Answer:**
**Directory Structure:**
```
terraform/
├── modules/
│   ├── vpc/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── compute/
│   └── database/
├── environments/
│   ├── dev/
│   │   ├── us-east-1/
│   │   │   ├── main.tf
│   │   │   ├── terraform.tfvars
│   │   │   └── backend.tf
│   │   └── eu-west-1/
│   ├── staging/
│   └── prod/
└── shared/
    ├── data.tf
    └── locals.tf
```

**Module Design Pattern:**
```hcl
# modules/vpc/main.tf
resource "aws_vpc" "main" {
  cidr_block           = var.cidr_block
  enable_dns_hostnames = var.enable_dns_hostnames
  enable_dns_support   = var.enable_dns_support

  tags = merge(var.common_tags, {
    Name = "${var.name_prefix}-vpc"
  })
}

# modules/vpc/variables.tf
variable "cidr_block" {
  description = "CIDR block for VPC"
  type        = string
  validation {
    condition     = can(cidrhost(var.cidr_block, 0))
    error_message = "Must be a valid CIDR block."
  }
}

# environments/prod/us-east-1/main.tf
module "vpc" {
  source = "../../../modules/vpc"
  
  cidr_block   = "10.0.0.0/16"
  name_prefix  = "prod-us-east-1"
  common_tags  = local.common_tags
}
```

**Problem**: Module versioning and dependency management
**Solution**: Use module versioning and registry:
```hcl
module "vpc" {
  source  = "app.terraform.io/company/vpc/aws"
  version = "~> 2.1"
}
```

### Q6: How do you handle cross-module dependencies and data sharing?

**Answer:**
**Methods:**

1. **Data Sources:**
```hcl
# In compute module
data "terraform_remote_state" "vpc" {
  backend = "s3"
  config = {
    bucket = "terraform-state-bucket"
    key    = "vpc/terraform.tfstate"
    region = "us-east-1"
  }
}

resource "aws_instance" "app" {
  subnet_id = data.terraform_remote_state.vpc.outputs.private_subnet_ids[0]
}
```

2. **Output/Input Pattern:**
```hcl
# VPC module outputs
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}

# Root module
module "vpc" {
  source = "./modules/vpc"
}

module "compute" {
  source = "./modules/compute"
  vpc_id = module.vpc.vpc_id
}
```

3. **Locals for Data Processing:**
```hcl
locals {
  availability_zones = data.aws_availability_zones.available.names
  private_subnets = [
    for i, az in local.availability_zones : cidrsubnet(var.vpc_cidr, 8, i)
  ]
}
```

**Anti-Pattern to Avoid:**
Tight coupling between modules through direct resource references.

## Performance and Optimization

### Q7: How do you optimize Terraform performance for large infrastructures?

**Answer:**
**Optimization Strategies:**

1. **Parallelism Control:**
```bash
terraform apply -parallelism=20
```

2. **Targeted Operations:**
```bash
terraform apply -target=module.vpc
terraform apply -target=aws_instance.web[0]
```

3. **State File Optimization:**
```hcl
# Use separate state files for different components
terraform {
  backend "s3" {
    bucket         = "terraform-state"
    key            = "networking/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-lock"
  }
}
```

4. **Resource Organization:**
```hcl
# Use for_each instead of count for better resource tracking
resource "aws_instance" "web" {
  for_each = var.instance_configs
  
  ami           = each.value.ami
  instance_type = each.value.instance_type
  
  tags = {
    Name = each.key
  }
}
```

**Performance Problem:**
Terraform taking too long to plan/apply with 1000+ resources.

**Solution:**
- Split infrastructure into logical state files
- Use `-refresh=false` for known stable environments
- Implement resource tagging for better filtering
- Use workspaces for environment separation

### Q8: Explain Terraform workspaces and when to use them vs. separate state files.

**Answer:**
**Workspaces:**
```bash
# Create and switch workspaces
terraform workspace new dev
terraform workspace new prod
terraform workspace select dev
```

**Implementation:**
```hcl
locals {
  environment = terraform.workspace
  instance_count = {
    dev  = 1
    prod = 3
  }
}

resource "aws_instance" "app" {
  count         = local.instance_count[local.environment]
  ami           = var.ami_id
  instance_type = local.environment == "prod" ? "t3.medium" : "t3.micro"
}
```

**When to Use Workspaces:**
- Same infrastructure, different environments
- Similar resource configurations
- Single team/repository

**When to Use Separate State Files:**
- Different infrastructure patterns
- Different teams/ownership
- Different security requirements
- Large-scale infrastructures

**Problem**: Workspace state conflicts
**Solution**: Use remote backend with workspace isolation:
```hcl
terraform {
  backend "s3" {
    bucket = "terraform-state"
    key    = "env:/${terraform.workspace}/terraform.tfstate"
    region = "us-east-1"
  }
}
```

## Advanced Terraform Features

### Q9: How do you implement custom providers and use provisioners effectively?

**Answer:**
**Custom Provider Development:**
```go
// Basic provider structure
func Provider() *schema.Provider {
    return &schema.Provider{
        Schema: map[string]*schema.Schema{
            "api_url": {
                Type:        schema.TypeString,
                Required:    true,
                DefaultFunc: schema.EnvDefaultFunc("API_URL", nil),
            },
        },
        ResourcesMap: map[string]*schema.Resource{
            "custom_resource": resourceCustomResource(),
        },
        ConfigureFunc: providerConfigure,
    }
}
```

**Provisioner Best Practices:**
```hcl
resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = "t3.micro"

  # Use provisioners as last resort
  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update",
      "sudo apt-get install -y nginx",
    ]

    connection {
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("~/.ssh/id_rsa")
      host        = self.public_ip
    }
  }

  # Always include destroy provisioner
  provisioner "local-exec" {
    when    = destroy
    command = "echo 'Instance ${self.id} destroyed'"
  }
}
```

**Problem**: Provisioner failures causing resource to be marked as tainted
**Solution**: Use null_resource for better control:
```hcl
resource "null_resource" "configuration" {
  triggers = {
    instance_id = aws_instance.web.id
  }

  provisioner "remote-exec" {
    # Configuration commands
  }

  depends_on = [aws_instance.web]
}
```

### Q10: How do you implement dynamic blocks and complex data transformations?

**Answer:**
**Dynamic Blocks:**
```hcl
resource "aws_security_group" "web" {
  name = "web-sg"

  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
      description = ingress.value.description
    }
  }

  dynamic "egress" {
    for_each = var.allow_all_egress ? [1] : []
    content {
      from_port   = 0
      to_port     = 0
      protocol    = "-1"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }
}
```

**Complex Data Transformations:**
```hcl
locals {
  # Flatten nested structures
  subnet_route_associations = flatten([
    for subnet_key, subnet in var.subnets : [
      for route_table in subnet.route_tables : {
        subnet_id      = aws_subnet.main[subnet_key].id
        route_table_id = aws_route_table.main[route_table].id
        association_key = "${subnet_key}-${route_table}"
      }
    ]
  ])

  # Transform to map for for_each
  subnet_route_map = {
    for assoc in local.subnet_route_associations :
    assoc.association_key => assoc
  }
}

resource "aws_route_table_association" "main" {
  for_each = local.subnet_route_map

  subnet_id      = each.value.subnet_id
  route_table_id = each.value.route_table_id
}
```

## CI/CD and Automation

### Q11: Design a complete CI/CD pipeline for Terraform with proper testing and security scanning.

**Answer:**
**Pipeline Structure:**
```yaml
# .github/workflows/terraform.yml
name: Terraform CI/CD

on:
  pull_request:
    paths:
    - 'terraform/**'
  push:
    branches:
    - main

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v2
      with:
        terraform_version: 1.5.0
    
    - name: Terraform Format Check
      run: terraform fmt -check -recursive
    
    - name: Terraform Init
      run: terraform init -backend=false
    
    - name: Terraform Validate
      run: terraform validate
    
    - name: Security Scan with Checkov
      run: |
        pip install checkov
        checkov -d . --framework terraform
    
    - name: Cost Estimation
      uses: infracost/infracost-gh-action@v0.16
      with:
        api_key: ${{ secrets.INFRACOST_API_KEY }}
        config_file: infracost.yml

  plan:
    needs: validate
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'
    steps:
    - name: Terraform Plan
      run: |
        terraform init
        terraform plan -out=tfplan
        terraform show -json tfplan > plan.json
    
    - name: Comment PR with Plan
      uses: actions/github-script@v6
      with:
        script: |
          const fs = require('fs');
          const plan = fs.readFileSync('plan.json', 'utf8');
          // Process and comment plan

  apply:
    needs: validate
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: production
    steps:
    - name: Terraform Apply
      run: |
        terraform init
        terraform apply -auto-approve
```

**Testing Strategy:**
```bash
# Unit tests with Terratest
func TestVPCModule(t *testing.T) {
    terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
        TerraformDir: "../modules/vpc",
        Vars: map[string]interface{}{
            "cidr_block": "10.0.0.0/16",
        },
    })

    defer terraform.Destroy(t, terraformOptions)
    terraform.InitAndApply(t, terraformOptions)

    vpcId := terraform.Output(t, terraformOptions, "vpc_id")
    assert.NotEmpty(t, vpcId)
}
```

### Q12: How do you handle secrets and credentials in CI/CD pipelines?

**Answer:**
**Best Practices:**

1. **OIDC Authentication:**
```yaml
permissions:
  id-token: write
  contents: read

steps:
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v2
  with:
    role-to-assume: arn:aws:iam::123456789012:role/GitHubActions
    aws-region: us-east-1
```

2. **Secret Management:**
```hcl
# Use external secret managers
data "aws_secretsmanager_secret_version" "db_credentials" {
  secret_id = "prod/database/credentials"
}

locals {
  db_creds = jsondecode(data.aws_secretsmanager_secret_version.db_credentials.secret_string)
}
```

3. **Environment-specific Configurations:**
```bash
# Use different backends per environment
terraform init \
  -backend-config="bucket=${BACKEND_BUCKET}" \
  -backend-config="key=${ENVIRONMENT}/terraform.tfstate"
```

## Troubleshooting and Debugging

### Q13: Walk through debugging a complex Terraform state corruption issue.

**Answer:**
**Scenario**: State file shows resources that don't exist, causing plan failures.

**Debugging Steps:**

1. **Analyze State:**
```bash
# Backup current state
cp terraform.tfstate terraform.tfstate.backup

# List resources in state
terraform state list

# Show specific resource
terraform state show aws_instance.web

# Check actual AWS resources
aws ec2 describe-instances --instance-ids i-1234567890abcdef0
```

2. **Identify Discrepancies:**
```bash
# Refresh state (use carefully)
terraform refresh

# Compare state with actual infrastructure
terraform plan -detailed-exitcode
```

3. **Resolution Options:**
```bash
# Option 1: Remove orphaned resources from state
terraform state rm aws_instance.orphaned

# Option 2: Import missing resources
terraform import aws_instance.missing i-0987654321fedcba0

# Option 3: Rebuild state (last resort)
terraform state rm $(terraform state list)
# Re-import all resources
```

**Prevention Strategy:**
```bash
# Automated state validation script
#!/bin/bash
set -e

echo "Validating state consistency..."
terraform plan -detailed-exitcode -out=validation.plan

if [ $? -eq 2 ]; then
    echo "State drift detected!"
    terraform show validation.plan
    exit 1
fi
echo "State is consistent"
```

### Q14: How do you handle circular dependencies and complex resource relationships?

**Answer:**
**Problem Example:**
```hcl
# This creates a circular dependency
resource "aws_security_group" "web" {
  ingress {
    from_port       = 443
    to_port         = 443
    protocol        = "tcp"
    security_groups = [aws_security_group.app.id]
  }
}

resource "aws_security_group" "app" {
  ingress {
    from_port       = 8080
    to_port         = 8080
    protocol        = "tcp"
    security_groups = [aws_security_group.web.id]
  }
}
```

**Solutions:**

1. **Use Security Group Rules:**
```hcl
resource "aws_security_group" "web" {
  name = "web-sg"
}

resource "aws_security_group" "app" {
  name = "app-sg"
}

resource "aws_security_group_rule" "web_to_app" {
  type                     = "ingress"
  from_port                = 8080
  to_port                  = 8080
  protocol                 = "tcp"
  security_group_id        = aws_security_group.app.id
  source_security_group_id = aws_security_group.web.id
}

resource "aws_security_group_rule" "app_to_web" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  security_group_id        = aws_security_group.web.id
  source_security_group_id = aws_security_group.app.id
}
```

2. **Use Depends_on for Complex Dependencies:**
```hcl
resource "aws_instance" "app" {
  # configuration

  depends_on = [
    aws_vpc.main,
    aws_security_group.app,
    aws_iam_role_policy_attachment.app
  ]
}
```

## Infrastructure as Code Best Practices

### Q15: Describe your approach to implementing infrastructure governance and compliance.

**Answer:**
**Governance Framework:**

1. **Policy as Code:**
```hcl
# Sentinel policy example
import "tfplan/v2" as tfplan

# Ensure all S3 buckets have encryption
encrypt_s3_buckets = rule {
    all tfplan.resource_changes as _, changes {
        changes.type is "aws_s3_bucket" and
        changes.change.after.server_side_encryption_configuration is not null
    }
}

main = rule {
    encrypt_s3_buckets
}
```

2. **Standardized Modules:**
```hcl
# Compliant S3 module
module "secure_s3_bucket" {
  source = "terraform-aws-modules/s3-bucket/aws"
  
  bucket = var.bucket_name
  
  versioning = {
    enabled = true
  }
  
  server_side_encryption_configuration = {
    rule = {
      apply_server_side_encryption_by_default = {
        sse_algorithm = "AES256"
      }
    }
  }
  
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

3. **Automated Compliance Checking:**
```yaml
# Pre-commit hooks
repos:
- repo: https://github.com/pre-commit/pre-commit-hooks
  hooks:
  - id: trailing-whitespace
  - id: end-of-file-fixer

- repo: https://github.com/gruntwork-io/pre-commit
  hooks:
  - id: terraform-fmt
  - id: terraform-validate
  - id: tflint

- repo: https://github.com/bridgecrewio/checkov
  hooks:
  - id: checkov
```

**Problem**: Teams bypassing governance controls
**Solution**: Implement mandatory CI/CD gates and use Terraform Cloud/Enterprise with policy enforcement.

## Final Advanced Scenarios

### Q16: You're tasked with migrating from on-premises to cloud while maintaining zero downtime. Describe your Terraform strategy.

**Answer:**
**Migration Strategy:**

1. **Hybrid Setup Phase:**
```hcl
# Create cloud infrastructure
module "aws_vpc" {
  source = "./modules/vpc"
  
  cidr_block = "10.1.0.0/16"
  
  # VPN connection to on-premises
  enable_vpn_gateway = true
  vpn_gateway_id     = aws_vpn_gateway.main.id
}

# Site-to-site VPN
resource "aws_vpn_connection" "main" {
  vpn_gateway_id      = aws_vpn_gateway.main.id
  customer_gateway_id = aws_customer_gateway.main.id
  type                = "ipsec.1"
  static_routes_only  = true
}
```

2. **Database Migration:**
```hcl
# Create RDS with replication from on-premises
resource "aws_db_instance" "replica" {
  identifier = "app-replica"
  
  # Configure as read replica initially
  replicate_source_db = "on-premises-source"
  
  # Later promote to standalone
  # backup_retention_period = 7
  # backup_window          = "03:00-04:00"
}
```

3. **Gradual Traffic Migration:**
```hcl
# Route53 weighted routing
resource "aws_route53_record" "app" {
  zone_id = data.aws_route53_zone.main.zone_id
  name    = "app.example.com"
  type    = "A"
  
  weighted_routing_policy {
    weight = var.cloud_traffic_weight # Start with 10%, gradually increase
  }
  
  alias {
    name                   = aws_lb.main.dns_name
    zone_id                = aws_lb.main.zone_id
    evaluate_target_health = true
  }
}
```

4. **Rollback Strategy:**
```hcl
# Maintain ability to rollback
locals {
  rollback_mode = var.enable_rollback
}

resource "aws_route53_record" "failover" {
  count = local.rollback_mode ? 1 : 0
  
  zone_id = data.aws_route53_zone.main.zone_id
  name    = "app.example.com"
  type    = "A"
  
  failover_routing_policy {
    type = "SECONDARY"
  }
  
  # Points back to on-premises
  records = [var.onprem_public_ip]
}
```

### Q17: How would you design disaster recovery with Terraform across multiple regions?

**Answer:**
**DR Architecture:**

1. **Multi-Region Setup:**
```hcl
# Primary region
module "primary_region" {
  source = "./modules/region"
  
  providers = {
    aws = aws.primary
  }
  
  region          = "us-east-1"
  is_primary      = true
  backup_schedule = "daily"
}

# DR region
module "dr_region" {
  source = "./modules/region"
  
  providers = {
    aws = aws.dr
  }
  
  region     = "us-west-2"
  is_primary = false
  
  # Smaller capacity for cost optimization
  instance_count = 1
  instance_type  = "t3.small"
}
```

2. **Cross-Region Replication:**
```hcl
# S3 cross-region replication
resource "aws_s3_bucket_replication_configuration" "replication" {
  role   = aws_iam_role.replication.arn
  bucket = aws_s3_bucket.primary.id

  rule {
    id     = "replicate_all"
    status = "Enabled"

    destination {
      bucket        = aws_s3_bucket.dr.arn
      storage_class = "STANDARD_IA"
    }
  }
}

# RDS cross-region automated backups
resource "aws_db_instance" "primary" {
  backup_retention_period   = 7
  backup_window            = "03:00-04:00"
  copy_tags_to_snapshot    = true
  
  # Enable automated backups to DR region
  enabled_cloudwatch_logs_exports = ["error", "general", "slow-query"]
}
```

3. **Automated Failover:**
```hcl
# Route53 health checks and failover
resource "aws_route53_health_check" "primary" {
  fqdn                            = "api.example.com"
  port                            = 443
  type                            = "HTTPS"
  resource_path                   = "/health"
  failure_threshold               = 3
  request_interval                = 30
  cloudwatch_alarm_region         = "us-east-1"
  cloudwatch_alarm_name           = "primary-region-health"
}

resource "aws_route53_record" "primary" {
  zone_id = data.aws_route53_zone.main.zone_id
  name    = "api.example.com"
  type    = "A"

  health_check_id = aws_route53_health_check.primary.id
  
  failover_routing_policy {
    type = "PRIMARY"
  }

  alias {
    name    = module.primary_region.load_balancer_dns_name
    zone_id = module.primary_region.load_balancer_zone_id
  }
}
```

