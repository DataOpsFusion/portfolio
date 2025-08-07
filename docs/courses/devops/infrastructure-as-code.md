# Infrastructure as Code

Stop clicking buttons and start managing infrastructure like a developer. Version control, code reviews, and automated deployments for your infrastructure.

## Why Infrastructure as Code?

### Problems with Manual Infrastructure
- **Inconsistency**: "It works on my machine" for infrastructure
- **No version control**: Can't track changes or rollback
- **Documentation drift**: Reality doesn't match documentation
- **Scaling issues**: Manual processes don't scale
- **Human error**: Mistakes happen when clicking buttons

### Benefits of IaC
- **Reproducible**: Same code = same infrastructure
- **Version controlled**: Track all changes
- **Collaborative**: Code reviews for infrastructure changes
- **Automated**: Deploy with confidence
- **Cost effective**: Destroy when not needed

## IaC Tools Comparison

### Terraform
**Best for**: Multi-cloud, complex infrastructures
- Declarative language (HCL)
- Strong state management
- Large provider ecosystem
- Plan before apply

### CloudFormation
**Best for**: AWS-only environments
- Native AWS integration
- JSON/YAML templates
- Stack-based management
- AWS support included

### Pulumi
**Best for**: Developers who prefer real programming languages
- Use Python, TypeScript, Go, C#
- Object-oriented approach
- Strong typing
- Familiar development patterns

### Ansible
**Best for**: Configuration management + some provisioning
- Agentless
- YAML playbooks
- Great for OS configuration
- Limited cloud resource support

## Terraform Deep Dive

### Basic Concepts

```hcl
# Provider configuration
provider "aws" {
  region = "us-west-2"
}

# Resource definition
resource "aws_instance" "web" {
  ami           = "ami-0c02fb55956c7d316"
  instance_type = "t2.micro"
  
  tags = {
    Name = "WebServer"
    Environment = "Development"
  }
}

# Output values
output "instance_ip" {
  value = aws_instance.web.public_ip
}
```

### Advanced Patterns

#### Modules for Reusability
```hcl
# modules/web-server/main.tf
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  # ... more configuration
}

# Using the module
module "web_server" {
  source        = "./modules/web-server"
  instance_type = "t2.small"
}
```

#### State Management
- Local state for development
- Remote state for production
- State locking with DynamoDB
- State encryption

#### Workspaces for Environments
```bash
# Create environments
terraform workspace new development
terraform workspace new staging
terraform workspace new production

# Switch between environments
terraform workspace select production
```

## Best Practices

### 1. Structure Your Code
```
terraform/
├── environments/
│   ├── dev/
│   ├── staging/
│   └── prod/
├── modules/
│   ├── vpc/
│   ├── compute/
│   └── database/
├── policies/
└── scripts/
```

### 2. Use Variables and Locals
```hcl
# Variables for inputs
variable "environment" {
  description = "Environment name"
  type        = string
}

# Locals for computed values
locals {
  common_tags = {
    Environment = var.environment
    Project     = "my-project"
    ManagedBy   = "terraform"
  }
}
```

### 3. Plan Before Apply
```bash
# Always review changes
terraform plan -out=tfplan

# Apply only after review
terraform apply tfplan
```

### 4. Use Remote State
```hcl
terraform {
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "prod/terraform.tfstate"
    region = "us-west-2"
    
    # State locking
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

## Hands-On Projects

### Project 1: Three-Tier Web Application
Build a complete web application infrastructure:
- VPC with public/private subnets
- Load balancer
- Auto Scaling Group
- RDS database
- S3 for static assets

### Project 2: Kubernetes Cluster
Deploy a production-ready Kubernetes cluster:
- EKS/GKE cluster
- Node groups with auto-scaling
- Ingress controllers
- Monitoring and logging

### Project 3: Multi-Cloud Setup
Deploy the same application across multiple clouds:
- Terraform modules for portability
- Cloud-specific optimizations
- Cross-cloud networking
- Disaster recovery setup

## Advanced Topics

### GitOps Workflows
- Infrastructure CI/CD pipelines
- Automated testing for infrastructure
- Policy as code with Sentinel/OPA
- Drift detection and remediation

### Security Best Practices
- Least privilege IAM policies
- Secret management
- Network security
- Compliance scanning

### Cost Optimization
- Resource tagging strategies
- Right-sizing instances
- Spot instances and reserved capacity
- Cost monitoring and alerts

---

*Infrastructure as Code isn't just about automation - it's about bringing software engineering practices to infrastructure management.*
