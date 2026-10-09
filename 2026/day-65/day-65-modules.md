Task 1: Understand Module Structure

1) A Terraform module is just a directory with `.tf` files. Create this structure:
      
        terraform-modules/
          main.tf                    # Root module -- calls child modules
          variables.tf               # Root variables
          outputs.tf                 # Root outputs
          providers.tf               # Provider config
          modules/
            ec2-instance/
              main.tf                # EC2 resource definition
              variables.tf           # Module inputs
              outputs.tf             # Module outputs
            security-group/
              main.tf                # Security group resource definition
              variables.tf           # Module inputs
              outputs.tf             # Module outputs

Create all the directories and empty files. This is the standard layout every Terraform project follows.

*Document*: What is the difference between a "root module" and a "child module"?

| Feature           | Root Module                                          | Child Module                                              |
|-------------------|------------------------------------------------------|-----------------------------------------------------------|
| Definition        | Main Terraform configuration                         | Reusable Terraform configuration called by another module |   
| Location          | Main project directory                               | Usually a subdirectory or external source                 |   
| Execution         | Terraform commands are usually run here              | Called through a `module` block                           |   
| Purpose           | Manages the overall infrastructure                   | Manages a specific part of the infrastructure             |   
| Variables         | Receives values from `.tfvars` files or other inputs | Receives values through module arguments                  |  
| Outputs           | Displays useful infrastructure information           | Exposes values to the calling module                      |  

---

Task 2: Build a Custom EC2 Module

Create `modules/ec2-instance/`:

1) `variables.tf` -- define inputs:

* `ami_id` (string)
* `instance_type` (string, default: `"t2.micro"`)
* `subnet_id` (string)
* `security_group_ids` (list of strings)
* `instance_name` (string)
* `tags` (map of strings, default: `{}`)

`modules/ec2-instance/variables.tf`


      variable "ami_id" {
        description = "AMI ID for the EC2 instance"
        type        = string
      }
      
      variable "instance_type" {
        description = "EC2 instance type"
        type        = string
        default     = "t2.micro"
      }
      
      variable "subnet_id" {
        description = "Subnet ID for the EC2 instance"
        type        = string
      }
      
      variable "security_group_ids" {
        description = "List of security group IDs"
        type        = list(string)
      }
      
      variable "instance_name" {
        description = "Name of the EC2 instance"
        type        = string
      }
      
      variable "tags" {
        description = "Additional tags for the EC2 instance"
        type        = map(string)
        default     = {}
      }

2) `main.tf` -- define the resource:

* `aws_instance` using all the variables
* Merge the Name tag with additional tags

`modules/ec2-instance/main.tf`
      
      resource "aws_instance" "my_instance" {
        ami                    = var.ami_id
        instance_type          = var.instance_type
        subnet_id              = var.subnet_id
        vpc_security_group_ids = var.security_group_ids
      
        tags = merge(
          var.tags,
          {
            Name = var.instance_name
          }
        )
      }

3) `outputs.tf` -- expose:

* `instance_id`
* `public_ip`
* `private_ip`

`modules/ec2-instance/outputs.tf`

      output "instance_id" {
        description = "ID of the EC2 instance"
        value       = aws_instance.my_instance.id
      }
      
      output "public_ip" {
        description = "Public IP address of the EC2 instance"
        value       = aws_instance.my_instance.public_ip
      }
      
      output "private_ip" {
        description = "Private IP address of the EC2 instance"
        value       = aws_instance.my_instance.private_ip
      }

Do NOT apply yet -- just write the module.

---

Task 3: Build a Custom Security Group Module

Create `modules/security-group/`:

1) `variables.tf` -- define inputs:

* `vpc_id` (string)
* `sg_name` (string)
* `ingress_ports` (list of numbers, default: `[22, 80]`)
* `tags` (map of strings, default: `{}`)

`modules/security-group/variables.tf`

      variable "vpc_id" {
        description = "ID of the VPC"
        type        = string
      }
      
      variable "sg_name" {
        description = "Name of the security group"
        type        = string
      }
      
      variable "ingress_ports" {
        description = "List of inbound ports to allow"
        type        = list(number)
        default     = [22, 80]
      }
      
      variable "tags" {
        description = "Additional tags for the security group"
        type        = map(string)
        default     = {}
      }

2) `main.tf` -- define the resource:

* `aws_security_group` in the given VPC
* Use `dynamic "ingress"` block to create rules from the `ingress_ports` list
* Allow all egress

`modules/security-group/main.tf`

      resource "aws_security_group" "my_security_group" {
        name        = var.sg_name
        description = "Security group managed by Terraform"
        vpc_id      = var.vpc_id
      
        dynamic "ingress" {
          for_each = var.ingress_ports
      
          content {
            from_port   = ingress.value
            to_port     = ingress.value
            protocol    = "tcp"
            cidr_blocks = ["0.0.0.0/0"]
            description = "Allow TCP port ${ingress.value}"
          }
        }
      
        egress {
          from_port   = 0
          to_port     = 0
          protocol    = "-1"
          cidr_blocks = ["0.0.0.0/0"]
        }
      
        tags = merge(
          var.tags,
          {
            Name = var.sg_name
          }
        )
      }

3) `outputs.tf` -- expose:

* `sg_id`

`modules/security-group/outputs.tf`
      
      output "sg_id" {
        description = "ID of the security group"
        value       = aws_security_group.my_security_group.id
      }

This is your first time using a `dynamic` block -- it loops over a list to generate repeated nested blocks.

---

Task 4: Call Your Modules from Root

In the root `main.tf`, wire everything together:

1) Create a VPC and subnet directly (or reuse your Day 62 config)

            resource "aws_vpc" "main" {
              cidr_block           = "10.0.0.0/16"
              enable_dns_support   = true
              enable_dns_hostnames = true
            
              tags = merge(local.common_tags, {
                Name = "terraweek-vpc"
              })
            }

            resource "aws_subnet" "public" {
              vpc_id                  = aws_vpc.main.id
              cidr_block              = "10.0.1.0/24"
              map_public_ip_on_launch = true
            
              tags = merge(local.common_tags, {
                Name = "terraweek-public-subnet"
              })
            }


2) Call the security group module:

            module "web_sg" {
              source        = "./modules/security-group"
              vpc_id        = aws_vpc.main.id
              sg_name       = "terraweek-web-sg"
              ingress_ports = [22, 80, 443]
              tags          = local.common_tags
            }

3) Call the EC2 module -- deploy two instances with different names using the same module:

            module "web_server" {
              source             = "./modules/ec2-instance"
              ami_id             = data.aws_ami.amazon_linux.id
              instance_type      = "t3.micro"
              subnet_id          = aws_subnet.public.id
              security_group_ids = [module.web_sg.sg_id]
              instance_name      = "terraweek-web"
              tags               = local.common_tags
            }
            
            module "api_server" {
              source             = "./modules/ec2-instance"
              ami_id             = data.aws_ami.amazon_linux.id
              instance_type      = "t3.micro"
              subnet_id          = aws_subnet.public.id
              security_group_ids = [module.web_sg.sg_id]
              instance_name      = "terraweek-api"
              tags               = local.common_tags
            }
4) Add root outputs that reference module outputs:

            output "web_server_ip" {
              value = module.web_server.public_ip
            }
            
            output "api_server_ip" {
              value = module.api_server.public_ip
            }      

5) Apply:

            terraform init    # Downloads/links the local modules
            terraform plan    # Should show all resources from both module calls
            terraform apply

*Verify*: Two EC2 instances running, same security group, different names. Check the AWS console.

      Yes it has same security group

`main.tf`

      terraform {
        required_version = ">= 1.5.0"
      
        required_providers {
          aws = {
            source  = "hashicorp/aws"
            version = "~> 5.0"
          }
        }
      }
      
      provider "aws" {
        region = var.region
      }
      
      locals {
        common_tags = {
          Project     = "TerraWeek"
          Environment = "dev"
          ManagedBy   = "Terraform"
        }
      }
      
      # Get the latest Amazon Linux 2023 AMI
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
      
      # 1. Create VPC
      resource "aws_vpc" "main" {
        cidr_block           = "10.0.0.0/16"
        enable_dns_support   = true
        enable_dns_hostnames = true
      
        tags = merge(local.common_tags, {
          Name = "terraweek-vpc"
        })
      }
      
      # 2. Create public subnet
      resource "aws_subnet" "public" {
        vpc_id                  = aws_vpc.main.id
        cidr_block              = "10.0.1.0/24"
        map_public_ip_on_launch = true
      
        tags = merge(local.common_tags, {
          Name = "terraweek-public-subnet"
        })
      }
      
      # 3. Internet gateway
      resource "aws_internet_gateway" "main" {
        vpc_id = aws_vpc.main.id
      
        tags = merge(local.common_tags, {
          Name = "terraweek-igw"
        })
      }
      
      # 4. Public route table
      resource "aws_route_table" "public" {
        vpc_id = aws_vpc.main.id
      
        route {
          cidr_block = "0.0.0.0/0"
          gateway_id = aws_internet_gateway.main.id
        }
      
        tags = merge(local.common_tags, {
          Name = "terraweek-public-rt"
        })
      }
      
      resource "aws_route_table_association" "public" {
        subnet_id      = aws_subnet.public.id
        route_table_id = aws_route_table.public.id
      }
      
      # 5. Call the security group module
      module "web_sg" {
        source = "./modules/security-group"
      
        vpc_id        = aws_vpc.main.id
        sg_name       = "terraweek-web-sg"
        ingress_ports = [22, 80, 443]
        tags          = local.common_tags
      }
      
      # 6. Deploy the web server
      module "web_server" {
        source = "./modules/ec2-instance"
      
        ami_id             = data.aws_ami.amazon_linux.id
        instance_type      = "t3.micro"
        subnet_id          = aws_subnet.public.id
        security_group_ids = [module.web_sg.sg_id]
        instance_name      = "terraweek-web"
        tags               = local.common_tags
      }
      
      # 7. Deploy the API server using the same module
      module "api_server" {
        source = "./modules/ec2-instance"
      
        ami_id             = data.aws_ami.amazon_linux.id
        instance_type      = "t3.micro"
        subnet_id          = aws_subnet.public.id
        security_group_ids = [module.web_sg.sg_id]
        instance_name      = "terraweek-api"
        tags               = local.common_tags
      }

---

Task 5: Use a Public Registry Module

Instead of building your own VPC from scratch, use the official module from the Terraform Registry.

1) Replace your hand-written VPC resources with:

            module "vpc" {
              source  = "terraform-aws-modules/vpc/aws"
              version = "~> 5.0"
            
              name = "terraweek-vpc"
              cidr = "10.0.0.0/16"
            
              azs             = ["us-west-2a", "us-west-2b"]
              public_subnets  = ["10.0.1.0/24", "10.0.2.0/24"]
              private_subnets = ["10.0.3.0/24", "10.0.4.0/24"]
            
              enable_nat_gateway = false
              enable_dns_hostnames = true
            
              tags = local.common_tags
            }

2) Update your EC2 and SG module calls to reference `module.vpc.vpc_id` and `module.vpc.public_subnets[0]`

            module "web_sg" {
              source = "./modules/security-group"
            
              vpc_id        = module.vpc.vpc_id
              sg_name       = "terraweek-web-sg"
              ingress_ports = [22, 80, 443]
              tags          = local.common_tags
            }


            module "web_server" {
              source = "./modules/ec2-instance"
            
              ami_id             = data.aws_ami.amazon_linux.id
              instance_type      = "t3.micro"
              subnet_id          = module.vpc.public_subnets[0]
              security_group_ids = [module.web_sg.sg_id]
              instance_name      = "terraweek-web"
              tags               = local.common_tags
            }

3) Run:

* terraform init     # Downloads the registry module

      terraform init
      Initializing the backend...
      
      Initializing modules...
      Downloading registry.terraform.io/terraform-aws-modules/vpc/aws 5.21.0 for vpc...
      - vpc in .terraform\modules\vpc
      
      Initializing provider plugins...
      - Reusing previous version of hashicorp/aws from the dependency lock file
      - Using previously-installed hashicorp/aws v5.100.0
      
      Terraform has been successfully initialized!

* terraform plan

      Plan: 20 to add, 0 to change, 8 to destroy.
      
      Changes to Outputs:
        ~ api_server_id     = "i-0fa2d278ec968e56a" -> (known after apply)
        ~ api_server_ip     = "54.185.195.119" -> (known after apply)
        ~ security_group_id = "sg-015fd181dc1502d72" -> (known after apply)
        ~ web_server_id     = "i-01bceccaaf543da41" -> (known after apply)
        ~ web_server_ip     = "44.248.206.160" -> (known after apply)

* terraform apply

      Outputs:
      
      api_server_id = "i-0f901c67a1ef01cd5"
      api_server_ip = ""
      security_group_id = "sg-0404621cf47288461"
      web_server_id = "i-00a34b7ee59d2c1e6"
      web_server_ip = ""

4) Compare: how many resources did the VPC module create vs your hand-written VPC from Day 62?

| Feature                       | Hand-written VPC                       | Registry VPC Module                            |
|-------------------------------|----------------------------------------|------------------------------------------------|
| VPC                           | Defined manually                       | Created by the module                          |
| Public subnets                | Defined manually                       | Two configured                                 |
| Private subnets               | Not necessarily included               | Two configured                                 |
| Internet gateway              | Defined manually                       | Managed by the module                          |
| Route tables and associations | Defined manually                       | Managed by the module                          |
| NAT gateway                   | Depends on your configuration          | Disabled in this task                          |
| Reusability                   | Requires more manual configuration     | Reusable and configurable                      |
| Resource count                | Depends on your original configuration | Depends on enabled features and module version |  


*Document*: Where does Terraform download registry modules to? Check `.terraform/modules/`.

---

Task 6: Module Versioning and Best Practices

1) Pin your registry module version explicitly:

* `version = "5.1.0"` -- exact version
* `version = "~> 5.0"` -- any 5.x version
* `version = ">= 5.0, < 6.0"` -- range

2) Run `terraform init -upgrade` to check for newer versions

* terraform init -upgrade

            Initializing the backend...
            
            Upgrading modules...
            Downloading registry.terraform.io/terraform-aws-modules/vpc/aws 5.1.0 for vpc...
            - vpc in .terraform\modules\vpc
            - web_server in modules\ec2-instance
            - api_server in modules\ec2-instance
            - web_sg in modules\security-group
            
            Initializing provider plugins...
            - Finding hashicorp/aws versions matching ">= 5.0.0, ~> 5.0"...
            - Using previously-installed hashicorp/aws v5.100.0
            
            Terraform has been successfully initialized!

3) Check the state to see how modules appear:

 * terraform state list

            data.aws_ami.amazon_linux
            module.api_server.aws_instance.my_instance
            module.vpc.aws_default_network_acl.this[0]
            module.vpc.aws_default_route_table.default[0]
            module.vpc.aws_default_security_group.this[0]
            module.vpc.aws_internet_gateway.this[0]
            module.vpc.aws_route.public_internet_gateway[0]
            module.vpc.aws_route_table.private[0]
            module.vpc.aws_route_table.private[1]
            module.vpc.aws_route_table.public[0]
            module.vpc.aws_route_table_association.private[0]
            module.vpc.aws_route_table_association.private[1]
            module.vpc.aws_route_table_association.public[0]
            module.vpc.aws_route_table_association.public[1]
            module.vpc.aws_subnet.private[0]
            module.vpc.aws_subnet.private[1]
            module.vpc.aws_subnet.public[0]
            module.vpc.aws_subnet.public[1]
            module.vpc.aws_vpc.this[0]
            module.web_server.aws_instance.my_instance
            module.web_sg.aws_security_group.my_security_group

Notice the `module.vpc.`, `module.web_server.`, `module.web_sg.` prefixes.

4) Destroy everything:

            terraform destroy

`main.tf`
      
      terraform {
        required_version = ">= 1.5.0"
      
        required_providers {
          aws = {
            source  = "hashicorp/aws"
            version = "~> 5.0"
          }
        }
      }
      
      provider "aws" {
        region = var.region
      }
      
      locals {
        common_tags = {
          Project     = "TerraWeek"
          Environment = "dev"
          ManagedBy   = "Terraform"
        }
      }
      
      # Get the latest Amazon Linux 2023 AMI
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
      
      # 1. VPC using the Terraform Registry module
      module "vpc" {
        source  = "terraform-aws-modules/vpc/aws"
        version = "5.1.0"
      
        name = "terraweek-vpc"
        cidr = "10.0.0.0/16"
      
        azs             = ["us-west-2a", "us-west-2b"]
        public_subnets  = ["10.0.1.0/24", "10.0.2.0/24"]
        private_subnets = ["10.0.3.0/24", "10.0.4.0/24"]
      
        enable_nat_gateway   = false
        enable_dns_hostnames = true
      
        tags = local.common_tags
      }
      
      # 2. Security Group using your custom module
      module "web_sg" {
        source = "./modules/security-group"
      
        vpc_id        = module.vpc.vpc_id
        sg_name       = "terraweek-web-sg"
        ingress_ports = [22, 80, 443]
      
        tags = local.common_tags
      }
      
      # 3. Web server using your custom EC2 module
      module "web_server" {
        source = "./modules/ec2-instance"
      
        ami_id             = data.aws_ami.amazon_linux.id
        instance_type      = "t3.micro"
        subnet_id          = module.vpc.public_subnets[0]
        security_group_ids = [module.web_sg.sg_id]
        instance_name      = "terraweek-web"
      
        tags = local.common_tags
      }
      
      # 4. API server using the same EC2 module
      module "api_server" {
        source = "./modules/ec2-instance"
      
        ami_id             = data.aws_ami.amazon_linux.id
        instance_type      = "t3.micro"
        subnet_id          = module.vpc.public_subnets[0]
        security_group_ids = [module.web_sg.sg_id]
        instance_name      = "terraweek-api"
      
        tags = local.common_tags
      }

Document: Write down five module best practices:

* Always pin versions for registry modules

      Use explicit version constraints to avoid unexpected changes.

* Keep modules focused -- one concern per module

      Design each module around one responsibility, such as VPC, EC2, or security groups.

* Use variables for everything, hardcode nothing

      Avoid hardcoding values that should vary between environments or deployments.

* Always define outputs so callers can reference resources

      Expose resource IDs, IP addresses, and other values that calling modules need.

* Add a README.md to every custom module

      Document the module's purpose, inputs, outputs, and usage examples.
