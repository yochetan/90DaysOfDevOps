Task 1: Project Setup

1) Create a new project directory with proper file structure:

        terraform-eks/
          providers.tf        # Provider and backend config
          vpc.tf              # VPC module call
          eks.tf              # EKS module call
          variables.tf        # All input variables
          outputs.tf          # Cluster outputs
          terraform.tfvars    # Variable values

In `providers.tf`:

1) Pin the AWS provider to `~> 5.0`

        aws = {
              source  = "hashicorp/aws"
              version = "~> 5.0"
            }

2) Pin the Kubernetes provider (you will need it later)
        
        kubernetes = {
              source  = "hashicorp/kubernetes"
              version = "~> 2.0"
            }

3) Set your region

        provider "aws" {
          region = var.region
        }

`providers.tf`

        terraform {
          required_version = ">= 1.5.0"
        
          required_providers {
            aws = {
              source  = "hashicorp/aws"
              version = "~> 5.0"
            }
        
            kubernetes = {
              source  = "hashicorp/kubernetes"
              version = "~> 2.0"
            }
          }
        }
        
        provider "aws" {
          region = var.region
        }

In `variables.tf`, define:

* `region` (string)
* `cluster_name` (string, default: `"terraweek-eks"`)
* `cluster_version` (string, default: `"1.31"`)
* `node_instance_type` (string, default: `"t3.micro"`)
* `node_desired_count` (number, default: `2`)
* `vpc_cidr` (string, default: `"10.0.0.0/16"`)

`variables.tf`

        variable "region" {
          description = "AWS region for EKS infrastructure"
          type        = string
        }
        
        variable "cluster_name" {
          description = "Name of the EKS cluster"
          type        = string
          default     = "terraweek-eks"
        }
        
        variable "cluster_version" {
          description = "Kubernetes version for EKS"
          type        = string
          default     = "1.31"
        }
        
        variable "node_instance_type" {
          description = "EC2 instance type for EKS worker nodes"
          type        = string
          default     = "t3.micro"
        }
        
        variable "node_desired_count" {
          description = "Desired number of worker nodes"
          type        = number
          default     = 2
        }
        
        variable "vpc_cidr" {
          description = "CIDR block for the VPC"
          type        = string
          default     = "10.0.0.0/16"
        }

---

Task 2: Create the VPC with Registry Module

EKS requires a VPC with both public and private subnets across multiple availability zones.

In `vpc.tf`, use the `terraform-aws-modules/vpc/aws` module:

1) CIDR: `var.vpc_cidr`

2) At least 2 availability zones

3) 2 public subnets and 2 private subnets

4) Enable NAT gateway (single NAT to save cost): `enable_nat_gateway = true`, `single_nat_gateway = true`

5) Enable DNS hostnames: `enable_dns_hostnames = true`

6) Add the required EKS tags on subnets:

        public_subnet_tags = {
          "kubernetes.io/role/elb" = 1
        }
        
        private_subnet_tags = {
          "kubernetes.io/role/internal-elb" = 1
        }

Run `terraform init` and `terraform plan` to verify the VPC config before moving on.

*Document*: Why does EKS need both public and private subnets? What do the subnet tags do?
