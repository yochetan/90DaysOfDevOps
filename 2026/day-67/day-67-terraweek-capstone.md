Task 1: Learn Terraform Workspaces

1) Before building the project, understand workspaces:

        mkdir terraweek-capstone && cd terraweek-capstone
        terraform init
        
        # See current workspace
        terraform workspace show                    # default

        default

        # Create new workspaces
        terraform workspace new dev

        Created and switched to workspace "dev"!

        terraform workspace new staging

        Created and switched to workspace "staging"!

        terraform workspace new prod

        Created and switched to workspace "prod"!

        # List all workspaces
        terraform workspace list

          default
          dev
        * prod
          staging

        # Switch between them
        terraform workspace select dev

        Switched to workspace "dev".

        terraform workspace select staging

        Switched to workspace "staging".

        terraform workspace select prod

        Switched to workspace "prod".

Answer:

1) What does `terraform.workspace` return inside a config?

        terraform.workspace is a built-in Terraform expression that returns the name of the currently selected workspace.

2) Where does each workspace store its state file?

| Workspace | State file location                           |
|-----------|-----------------------------------------------|
| default   | terraform.tfstate                             |
| dev       | terraform.tfstate.d/dev/terraform.tfstate     |
| staging   | terraform.tfstate.d/staging/terraform.tfstate |
| prod      | terraform.tfstate.d/prod/terraform.tfstate    |

3) How is this different from using separate directories per environment?

| Feature                   | Terraform workspaces                                        | Separate directories                                            |
|---------------------------|-------------------------------------------------------------|-----------------------------------------------------------------|
| Configuration             | Same configuration files                                    | Each environment can have its own files                         |
| State                     | Separate state per workspace                                | Separate state per directory/backend configuration              |
| Code duplication          | Usually less                                                | Can be more if code is duplicated                               |
| Environment customization | Uses variables and workspace conditions                     | Easy to customize each environment independently                |
| Risk of mistakes          | Possible to select the wrong workspace                      | Possible to run Terraform in the wrong directory                |
| Best use                  | Similar environments with the same infrastructure structure | Environments needing different configurations or infrastructure |

---

Task 2: Set Up the Project Structure

Create this layout:
        
        terraweek-capstone/
          main.tf                   # Root module -- calls child modules
          variables.tf              # Root variables
          outputs.tf                # Root outputs
          providers.tf              # AWS provider and backend
          locals.tf                 # Local values using workspace
          dev.tfvars                # Dev environment values
          staging.tfvars            # Staging environment values
          prod.tfvars               # Prod environment values
          .gitignore                # Ignore state, .terraform, tfvars with secrets
          modules/
            vpc/
              main.tf
              variables.tf
              outputs.tf
            security-group/
              main.tf
              variables.tf
              outputs.tf
            ec2-instance/
              main.tf
              variables.tf
              outputs.tf

Create the `.gitignore`:

        .terraform/
        *.tfstate
        *.tfstate.backup
        *.tfvars
        .terraform.lock.hcl

*Document*: Why is this file structure considered best practice?

        This file structure is considered best practice because it makes your Terraform project modular, reusable, organized, and easier to maintain across multiple environments such as dev, staging, and prod.

---

Task 3: Build the Custom Modules

Create three focused modules:

Module 1: `modules/vpc/`

* Input: `cidr`, `public_subnet_cidr`, `environment`, `project_name`

`variables.tf`

        variable "cidr" {
          description = "CIDR block for the VPC"
          type        = string
        }
        
        variable "public_subnet_cidr" {
          description = "CIDR block for the public subnet"
          type        = string
        }
        
        variable "environment" {
          description = "Deployment environment"
          type        = string
        }
        
        variable "project_name" {
          description = "Name of the project"
          type        = string
        }

* Resources: VPC, public subnet, internet gateway, route table, route table association

`main.tf`

        locals {
          common_tags = {
            Environment = var.environment
            Project     = var.project_name
            ManagedBy   = "Terraform"
          }
        }
        
        resource "aws_vpc" "this" {
          cidr_block           = var.cidr
          enable_dns_support   = true
          enable_dns_hostnames = true
        
          tags = merge(local.common_tags, {
            Name = "${var.project_name}-${var.environment}-vpc"
          })
        }
        
        resource "aws_subnet" "public" {
          vpc_id                  = aws_vpc.this.id
          cidr_block              = var.public_subnet_cidr
          map_public_ip_on_launch = true
        
          tags = merge(local.common_tags, {
            Name = "${var.project_name}-${var.environment}-public-subnet"
          })
        }
        
        resource "aws_internet_gateway" "this" {
          vpc_id = aws_vpc.this.id
        
          tags = merge(local.common_tags, {
            Name = "${var.project_name}-${var.environment}-igw"
          })
        }
        
        resource "aws_route_table" "public" {
          vpc_id = aws_vpc.this.id
        
          route {
            cidr_block = "0.0.0.0/0"
            gateway_id = aws_internet_gateway.this.id
          }
        
          tags = merge(local.common_tags, {
            Name = "${var.project_name}-${var.environment}-public-rt"
          })
        }
        
        resource "aws_route_table_association" "public" {
          subnet_id      = aws_subnet.public.id
          route_table_id = aws_route_table.public.id
        }

* Output: `vpc_id`, `subnet_id`

`outputs.tf`

        output "vpc_id" {
          description = "ID of the created VPC"
          value       = aws_vpc.this.id
        }
        
        output "subnet_id" {
          description = "ID of the public subnet"
          value       = aws_subnet.public.id
        }

* All resources tagged with environment and project name

Module 2: `modules/security-group/`

* Input: `vpc_id`, `ingress_ports`, `environment`, `project_name`

`variables.tf`

        variable "vpc_id" {
          description = "ID of the VPC where the security group will be created"
          type        = string
        }
        
        variable "ingress_ports" {
          description = "TCP ports to allow inbound"
          type        = list(number)
        }
        
        variable "environment" {
          description = "Deployment environment"
          type        = string
        }
        
        variable "project_name" {
          description = "Name of the project"
          type        = string
        }

* Resources: Security group with dynamic ingress rules, allow all egress

`main.tf`

        resource "aws_security_group" "this" {
          name        = "${var.project_name}-${var.environment}-sg"
          description = "Security group for ${var.project_name}"
          vpc_id      = var.vpc_id
        
          dynamic "ingress" {
            for_each = var.ingress_ports
        
            content {
              description = "Allow TCP port ${ingress.value}"
              from_port   = ingress.value
              to_port     = ingress.value
              protocol    = "tcp"
              cidr_blocks = ["0.0.0.0/0"]
            }
          }
        
          egress {
            description = "Allow all outbound traffic"
            from_port   = 0
            to_port     = 0
            protocol    = "-1"
            cidr_blocks = ["0.0.0.0/0"]
          }
        
          tags = {
            Name        = "${var.project_name}-${var.environment}-sg"
            Environment = var.environment
            Project     = var.project_name
            ManagedBy   = "Terraform"
          }
        }

* Output: `sg_id`

`outputs.tf`
        
        output "sg_id" {
          description = "ID of the created security group"
          value       = aws_security_group.this.id
        }

Module 3: `modules/ec2-instance/`

* Input: `ami_id`, `instance_type`, `subnet_id`, `security_group_ids`, `environment`, `project_name`

`variables.tf`
        
        variable "ami_id" {
          description = "AMI ID for the EC2 instance"
          type        = string
        }
        
        variable "instance_type" {
          description = "EC2 instance type"
          type        = string
          default     = "t3.micro"
        }
        
        variable "subnet_id" {
          description = "ID of the subnet to launch the instance in"
          type        = string
        }
        
        variable "security_group_ids" {
          description = "List of security group IDs"
          type        = list(string)
        }
        
        variable "environment" {
          description = "Deployment environment"
          type        = string
        }
        
        variable "project_name" {
          description = "Name of the project"
          type        = string
        }

* Resources: EC2 instance with tags

`main.tf`

        resource "aws_instance" "this" {
          ami                         = var.ami_id
          instance_type               = var.instance_type
          subnet_id                   = var.subnet_id
          vpc_security_group_ids      = var.security_group_ids
          associate_public_ip_address = true
        
          tags = {
            Name        = "${var.project_name}-${var.environment}-ec2"
            Environment = var.environment
            Project     = var.project_name
            ManagedBy   = "Terraform"
          }
        }

* Output: `instance_id`, `public_ip`

`outputs.tf`

        output "instance_id" {
          description = "ID of the created EC2 instance"
          value       = aws_instance.this.id
        }
        
        output "public_ip" {
          description = "Public IPv4 address of the EC2 instance"
          value       = aws_instance.this.public_ip
        }

Write and validate each module:

- terraform validate

        Success! The configuration is valid.

---

Task 4: Wire It All Together with Workspace-Aware Config

In the root module, use `terraform.workspace` to drive environment-specific behavior.

`locals.tf`:

        locals {
          environment = terraform.workspace
          name_prefix = "${var.project_name}-${local.environment}"
        
          common_tags = {
            Project     = var.project_name
            Environment = local.environment
            ManagedBy   = "Terraform"
            Workspace   = terraform.workspace
          }
        }

`variables.tf`:

        variable "project_name" {
          type    = string
          default = "terraweek"
        }
        
        variable "vpc_cidr" {
          type = string
        }
        
        variable "subnet_cidr" {
          type = string
        }
        
        variable "instance_type" {
          type = string
        }
        
        variable "ingress_ports" {
          type    = list(number)
          default = [22, 80]
        }

`main.tf` -- call all three modules, passing workspace-aware names and variables.

        data "aws_ami" "amazon_linux" {
          most_recent = true
          owners      = ["amazon"]
        
          filter {
            name   = "name"
            values = ["al2023-ami-2023.*-x86_64"]
          }
        
          filter {
            name   = "architecture"
            values = ["x86_64"]
          }
        
          filter {
            name   = "virtualization-type"
            values = ["hvm"]
          }
        }
        
        module "vpc" {
          source = "./modules/vpc"
        
          cidr               = var.vpc_cidr
          public_subnet_cidr = var.subnet_cidr
          environment        = local.environment
          project_name       = var.project_name
        }
        
        module "security_group" {
          source = "./modules/security-group"
        
          vpc_id       = module.vpc.vpc_id
          ingress_ports = var.ingress_ports
          environment  = local.environment
          project_name = var.project_name
        }
        
        module "ec2_instance" {
          source = "./modules/ec2-instance"
        
          ami_id             = data.aws_ami.amazon_linux.id
          instance_type      = var.instance_type
          subnet_id          = module.vpc.subnet_id
          security_group_ids = [module.security_group.sg_id]
          environment        = local.environment
          project_name       = var.project_name
        }

Environment-specific tfvars:

`dev.tfvars`:

        vpc_cidr      = "10.0.0.0/16"
        subnet_cidr   = "10.0.1.0/24"
        instance_type = "t3.micro"
        ingress_ports = [22, 80]

`staging.tfvars`:

        vpc_cidr      = "10.1.0.0/16"
        subnet_cidr   = "10.1.1.0/24"
        instance_type = "t3.small"
        ingress_ports = [22, 80, 443]

`prod.tfvars`:

        vpc_cidr      = "10.2.0.0/16"
        subnet_cidr   = "10.2.1.0/24"
        instance_type = "t3.small"
        ingress_ports = [80, 443]

Notice: dev allows SSH, prod does not. Different CIDRs prevent overlap. Instance types scale up per environment.

---

Task 5: Deploy All Three Environments

Deploy each environment using its workspace and tfvars file:

Dev:

- terraform workspace select dev

        Switched to workspace "dev".

- terraform plan -var-file="dev.tfvars"

        Plan: 7 to add, 0 to change, 0 to destroy.

- terraform apply -var-file="dev.tfvars"

        Apply complete! Resources: 7 added, 0 changed, 0 destroyed.

Staging:

- terraform workspace select staging

        Switched to workspace "staging".

- terraform plan -var-file="staging.tfvars"

        Plan: 7 to add, 0 to change, 0 to destroy.

- terraform apply -var-file="staging.tfvars"

        Apply complete! Resources: 7 added, 0 changed, 0 destroyed.

Prod:

- terraform workspace select prod

        Switched to workspace "prod".

- terraform plan -var-file="prod.tfvars"

        Plan: 7 to add, 0 to change, 0 to destroy.

- terraform apply -var-file="prod.tfvars"

        Apply complete! Resources: 7 added, 0 changed, 0 destroyed.

After all three are deployed, verify:

# Check each workspace's resources
- terraform workspace select dev && terraform output

        environment = "dev"
        instance_id = "i-0b5a41beec3be7e97"
        public_ip = "44.255.180.1"
        security_group_id = "sg-098a77405e28ac2f4"
        subnet_id = "subnet-0e5ec89be597fa884"
        vpc_id = "vpc-0ee20f6058008518d"

- terraform workspace select staging && terraform output

        environment = "staging"
        instance_id = "i-0aeced8daa2d8e332"
        public_ip = "35.82.33.234"
        security_group_id = "sg-075d348371f6f22ee"
        subnet_id = "subnet-030b79f1373a7486e"
        vpc_id = "vpc-0f53561bf4ee2c3b3"

- terraform workspace select prod && terraform output

        environment = "prod"
        instance_id = "i-023cbfb54235c3ed3"
        public_ip = "52.41.119.219"
        security_group_id = "sg-035f197839b9d1632"
        subnet_id = "subnet-003ec2f0f279b843a"
        vpc_id = "vpc-0a75826a0d7dbfc71"

Go to the AWS console and verify:

* Three separate VPCs with different CIDR ranges

        yes

* Three EC2 instances with different instance types

        yes

* Different Name tags per environment: `terraweek-dev-server`, `terraweek-staging-server`, `terraweek-prod-server`

        yes

Verify: Are all three environments completely isolated from each other?

        No — they are separated, but not completely isolated.

| Isolation area              | Are they isolated? | Explanation                                                   |
|-----------------------------|--------------------|---------------------------------------------------------------|
| Terraform state             | Yes, logically     | Each workspace has its own state                              |
| VPC networks                | Yes, by default    | Each environment has its own VPC and non-overlapping CIDR     |
| EC2 instances               | Separate resources | Each environment has its own instance                         |
| AWS account and permissions | Not necessarily    | All three may use the same AWS account and IAM permissions    |
| Security                    | Partially          | Security groups differ, but dev and staging permit public SSH |
| Costs and quotas            | No                 | All resources can consume the same accounts budget and quotas |

---

Task 6: Document Best Practices

Write down everything you have learned this week as a Terraform best practices guide:

1) File structure -- separate files for providers, variables, outputs, main, locals

2) State management -- always use remote backend, enable locking, enable versioning

3) Variables -- never hardcode, use tfvars per environment, validate with `validation` blocks

4) Modules -- one concern per module, always define inputs/outputs, pin registry module versions

5) Workspaces -- use for environment isolation, reference `terraform.workspace` in configs

6) Security -- .gitignore for state and tfvars, encrypt state at rest, restrict backend access

7) Commands -- always run `plan` before `apply`, use `fmt` and `validate` before committing

8) Tagging -- tag every resource with project, environment, and managed-by

9) Naming -- consistent prefix pattern: `<project>-<environment>-<resource>`

10) Cleanup -- always `terraform destroy` non-production environments when not in use

---

Task 7: Destroy All Environments

Clean up all three environments in reverse order:

- terraform workspace select prod

        Switched to workspace "prod".

- terraform destroy -var-file="prod.tfvars"

        Destroy complete! Resources: 7 destroyed.

- terraform workspace select staging

        Switched to workspace "staging".

- terraform destroy -var-file="staging.tfvars"

        Destroy complete! Resources: 7 destroyed.

- terraform workspace select dev

        Switched to workspace "dev".

- terraform destroy -var-file="dev.tfvars"

        Destroy complete! Resources: 7 destroyed.

Verify in the AWS console -- all VPCs, instances, security groups, and gateways should be gone.

Delete the workspaces:

- terraform workspace select default

        Switched to workspace "default".

- terraform workspace delete dev

        Deleted workspace "dev"!

- terraform workspace delete staging

        Deleted workspace "staging"!

- terraform workspace delete prod

        Deleted workspace "prod"!

Verify: Is your AWS account completely clean?

        Yes
