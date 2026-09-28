Task 1: Explore the AWS Provider

1) Create a new project directory: `terraform-aws-infra`

        Created!!

2) Write a `providers.tf` file:

- Define the `terraform` block with `required_providers` pinning the AWS provider to version `~> 5.0`

        terraform {
          required_providers {
                aws = {
                    source = "hashicorp/aws"
                    version = "~> 5.0"
                }
          }
        }

- Define the `provider "aws"` block with your region

        provider "aws" {
          region = "us-west-2"
        }

3) Run `terraform init` and check the output -- what version was installed?

- terraform init

        Initializing the backend...
        
        Initializing provider plugins...
        - Finding hashicorp/aws versions matching "~> 5.0"...
        - Installing hashicorp/aws v5.100.0...
        - Installed hashicorp/aws v5.100.0 (signed by HashiCorp)
        
4) Read the provider lock file `.terraform.lock.hcl` -- what does it do?

        The purpose is to make provider installations consistent and verifiable across machines and future terraform init runs.

Document: What does `~> 5.0` mean? How is it different from `>= 5.0` and `= 5.0.0`?

        This is called the pessimistic version constraint.
        ~> 5.0
        means:
        Use AWS provider version 5.x, but don't automatically move to version 6.x.

---

Task 2: Build a VPC from Scratch

Create a `main.tf` and define these resources one by one:

1) `aws_vpc` -- CIDR block `10.0.0.0/16`, tag it `"TerraWeek-VPC"`

        resource "aws_vpc" "main" {
          cidr_block = "10.0.0.0/16"
          tags = {
            Name = "TerraWeek-VPC"
          }
        }

2) `aws_subnet` -- CIDR block `10.0.1.0/24`, reference the VPC ID from step 1, enable public IP on launch, tag it `"TerraWeek-Public-Subnet"`

        resource "aws_subnet" "main" {
          vpc_id     = aws_vpc.main.id
          cidr_block = "10.0.1.0/24"
          map_public_ip_on_launch = true

          tags = {
            Name = "TerraWeek-Public-Subnet"
          }
        }

3) `aws_internet_gateway` -- attach it to the VPC

        resource "aws_internet_gateway" "gw" {
          vpc_id = aws_vpc.main.id
        
          tags = {
            Name = "main"
          }
        }

4) `aws_route_table` -- create it in the VPC, add a route for `0.0.0.0/0` pointing to the internet gateway

        resource "aws_route_table" "example" {
          vpc_id = aws_vpc.main.id
        
          route {
            cidr_block = "0.0.0.0/0"
            gateway_id = aws_internet_gateway.gw.id
          }
        
          tags = {
            Name = "example"
          }
        }

5) `aws_route_table_association` -- associate the route table with the subnet

        resource "aws_route_table_association" "example" {
          subnet_id      = aws_subnet.main.id
          route_table_id = aws_route_table.example.id
        }

Run `terraform plan` -- you should see 5 resources to create.

- terraform plan    

        Terraform used the selected providers to generate the following execution plan. Resource actions are indicated
        with the following symbols:
          + create
        
        Terraform will perform the following actions:
        
          # aws_internet_gateway.gw will be created
          + resource "aws_internet_gateway" "gw" {
              + arn      = (known after apply)
              + id       = (known after apply)
              + owner_id = (known after apply)
              + tags     = {
                  + "Name" = "main"
                }
              + tags_all = {
                  + "Name" = "main"
                }
              + vpc_id   = (known after apply)
            }
        
          # aws_route_table.example will be created
          + resource "aws_route_table" "example" {
              + arn              = (known after apply)
              + id               = (known after apply)
              + owner_id         = (known after apply)
              + propagating_vgws = (known after apply)
              + route            = [
                  + {
                      + cidr_block                 = "0.0.0.0/0"
                      + gateway_id                 = (known after apply)
                        # (11 unchanged attributes hidden)
                    },
                ]
              + tags             = {
                  + "Name" = "example"
                }
              + tags_all         = {
                  + "Name" = "example"
                }
              + vpc_id           = (known after apply)
            }
        
          # aws_route_table_association.example will be created
          + resource "aws_route_table_association" "example" {
              + id             = (known after apply)
              + route_table_id = (known after apply)
              + subnet_id      = (known after apply)
            }
        
          # aws_subnet.main will be created
          + resource "aws_subnet" "main" {
              + arn                                            = (known after apply)
              + assign_ipv6_address_on_creation                = false
              + availability_zone                              = (known after apply)
              + availability_zone_id                           = (known after apply)
              + cidr_block                                     = "10.0.1.0/24"
              + enable_dns64                                   = false
              + enable_resource_name_dns_a_record_on_launch    = false
              + enable_resource_name_dns_aaaa_record_on_launch = false
              + id                                             = (known after apply)
              + ipv6_cidr_block_association_id                 = (known after apply)
              + ipv6_native                                    = false
              + map_public_ip_on_launch                        = true
              + owner_id                                       = (known after apply)
              + private_dns_hostname_type_on_launch            = (known after apply)
              + tags                                           = {
                  + "Name" = "TerraWeek-Public-Subnet"
                }
              + tags_all                                       = {
                  + "Name" = "TerraWeek-Public-Subnet"
                }
              + vpc_id                                         = (known after apply)
            }
        
          # aws_vpc.main will be created
          + resource "aws_vpc" "main" {
              + arn                                  = (known after apply)
              + cidr_block                           = "10.0.0.0/16"
              + default_network_acl_id               = (known after apply)
              + default_route_table_id               = (known after apply)
              + default_security_group_id            = (known after apply)
              + dhcp_options_id                      = (known after apply)
              + enable_dns_hostnames                 = (known after apply)
              + enable_dns_support                   = true
              + enable_network_address_usage_metrics = (known after apply)
              + id                                   = (known after apply)
              + instance_tenancy                     = "default"
              + ipv6_association_id                  = (known after apply)
              + ipv6_cidr_block                      = (known after apply)
              + ipv6_cidr_block_network_border_group = (known after apply)
              + main_route_table_id                  = (known after apply)
              + owner_id                             = (known after apply)
              + tags                                 = {
                  + "Name" = "TerraWeek-VPC"
                }
              + tags_all                             = {
                  + "Name" = "TerraWeek-VPC"
                }
            }
        
        Plan: 5 to add, 0 to change, 0 to destroy.

Verify: Apply and check the AWS VPC console. Can you see all five resources connected?



---

Task 3: Understand Implicit Dependencies

Look at your `main.tf` carefully:

1) The subnet references `aws_vpc.main.id` -- this is an implicit dependency

2) The internet gateway references the VPC ID -- another implicit dependency

3) The route table association references both the route table and the subnet

Answer these questions:

- How does Terraform know to create the VPC before the subnet?

        Because your subnet contains:
        
        vpc_id = aws_vpc.main.id
        
        Terraform sees that `aws_subnet.main` needs the ID of aws_vpc.main.
        
        So Terraform creates a dependency:
        
        aws_vpc.main
              ↓
        aws_subnet.main
        
        Terraform doesn't simply execute main.tf from top to bottom. It analyzes resource references and builds a dependency graph.
        
        Therefore, even if you wrote the subnet resource before the VPC resource in the file, Terraform would still know that the VPC needs to be created first.

- What would happen if you tried to create the subnet before the VPC existed?

- Find all implicit dependencies in your config and list them

