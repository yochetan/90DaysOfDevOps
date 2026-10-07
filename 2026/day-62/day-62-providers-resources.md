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

        terraform apply -auto-approve
        Apply complete! Resources: 5 added, 0 changed, 0 destroyed.

        Outputs:
        
        instance_id = "i-0d4dd5395e66079c2"
        instance_public_dns = ""
        instance_public_ip = "34.213.90.43"
        security_group_id = "sg-0fa673d9571191196"
        subnet_id = "subnet-0f4bbe093366313a8"
        vpc_id = "vpc-028e86a077f419eaa"

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
        
        Terraform sees that aws_subnet.main needs the ID of aws_vpc.main.
        
        So Terraform creates a dependency:
        
                aws_vpc.main
                      ↓
                aws_subnet.main
                
        Terraform doesn't simply execute main.tf from top to bottom. It analyzes resource references and builds a dependency graph.
        
        Therefore, even if you wrote the subnet resource before the VPC resource in the file, Terraform would still know that the VPC needs to be created first.

- What would happen if you tried to create the subnet before the VPC existed?

        If you mean the VPC resource hasn't been created yet, Terraform will still handle it correctly because of:
        
                vpc_id = aws_vpc.main.id
        
        Terraform knows:
        
        "I need the VPC ID before I can create this subnet."
        
        So it creates:
        
                VPC
                 ↓
                Subnet
        
        You don't need to manually create the VPC first.
        
        However
        
        If you tried to reference a VPC that doesn't exist in your Terraform configuration or AWS, Terraform would fail because it couldn't obtain a valid VPC ID.
        
        For example, something like:
        
                vpc_id = "some-vpc-that-does-not-exist"
        
        could result in an AWS error when Terraform tries to create the subnet.

- Find all implicit dependencies in your config and list them

* Dependency 1 — Subnet → VPC

        resource "aws_subnet" "main" {
          vpc_id = aws_vpc.main.id
        }

Dependency:

        aws_vpc.main
              ↓
        aws_subnet.main

* Dependency 2 — Internet Gateway → VPC

        resource "aws_internet_gateway" "gw" {
          vpc_id = aws_vpc.main.id
        }

Dependency:
        
        aws_vpc.main
              ↓
        aws_internet_gateway.gw

* Dependency 3 — Route Table → VPC

        resource "aws_route_table" "example" {
          vpc_id = aws_vpc.main.id
        }

Dependency:

        aws_vpc.main
              ↓
        aws_route_table.example

* Dependency 4 — Route Table → Internet Gateway

Inside the route:

        gateway_id = aws_internet_gateway.gw.id

Dependency:

        aws_internet_gateway.gw
              ↓
        aws_route_table.example

This is important because the route needs the Internet Gateway ID.

* Dependency 5 — Route Table Association → Subnet

        subnet_id = aws_subnet.main.id

Dependency:
        
        aws_subnet.main
              ↓
        aws_route_table_association.example

* Dependency 6 — Route Table Association → Route Table

        route_table_id = aws_route_table.example.id

Dependency:

        aws_route_table.example
              ↓
        aws_route_table_association.example

---

Task 4: Add a Security Group and EC2 Instance

Add to your config:

1) `aws_security_group` in the VPC:

- Ingress rule: allow SSH (port 22) from `0.0.0.0/0`

        ingress {
            description = "SSH"
            from_port   = 22
            to_port     = 22
            protocol    = "tcp"
            cidr_blocks = ["0.0.0.0/0"]
          }

- Ingress rule: allow HTTP (port 80) from `0.0.0.0/0`

        ingress {
            description = "HTTP"
            from_port   = 80
            to_port     = 80
            protocol    = "tcp"
            cidr_blocks = ["0.0.0.0/0"]
          }

- Egress rule: allow all outbound traffic

        egress {
            from_port   = 0
            to_port     = 0
            protocol    = "-1"
            cidr_blocks = ["0.0.0.0/0"]
          }

- Tag: `"TerraWeek-SG"`

        tags = {
            Name = "TerraWeek-SG"
          }

2) `aws_instance` in the subnet:

- Use Amazon Linux 2 AMI for your region

        data "aws_ami" "amazon_linux_2" {
          most_recent = true
        
          owners = ["amazon"]
        
          filter {
            name   = "name"
            values = ["amzn2-ami-hvm-*-x86_64-gp2"]
          }
        
          filter {
            name   = "state"
            values = ["available"]
          }
        }

- Instance type: `t2.micro`
- Associate the security group
- Set `associate_public_ip_address` = true
- Tag: `"TerraWeek-Server"`

          ami                         = data.aws_ami.amazon_linux_2.id
          instance_type               = "t2.micro"
          subnet_id                   = aws_subnet.main.id
          vpc_security_group_ids      = [aws_security_group.main.id]
          associate_public_ip_address = true
        
          tags = {
            Name = "TerraWeek-Server"
          }
Apply and verify -- your EC2 instance should have a public IP and be reachable.



---

Task 5: Explicit Dependencies with depends_on

Sometimes Terraform cannot detect a dependency automatically.

1) Add a second `aws_s3_bucket` resource for application logs

        resource "aws_s3_bucket" "app_logs" {
          bucket = "chetan-terraweek-app-logs-2026"
          depends_on = [aws_instance.main]
        
          tags = {
            Name = "TerraWeek-App-Logs"
          }
        }

2) Add `depends_on = [aws_instance.main]` to the S3 bucket -- even though there is no direct reference, you want the bucket created only after the instance

        depends_on = [aws_instance.main]

3) Run `terraform plan` and observe the order

- terraform plan    

        data.aws_ami.amazon_linux_2: Reading...
        data.aws_ami.amazon_linux_2: Read complete after 2s [id=ami-0a548888746409a3d]
        
        Terraform used the selected providers to generate the following execution plan. Resource actions are indicated
        with the following symbols:
          + create
        
        Terraform will perform the following actions:
        
          # aws_instance.main will be created
          + resource "aws_instance" "main" {
              + ami                                  = "ami-0a548888746409a3d"
              + arn                                  = (known after apply)
              + associate_public_ip_address          = true
              + availability_zone                    = (known after apply)
              + cpu_core_count                       = (known after apply)
              + cpu_threads_per_core                 = (known after apply)
              + disable_api_stop                     = (known after apply)
              + disable_api_termination              = (known after apply)
              + ebs_optimized                        = (known after apply)
              + enable_primary_ipv6                  = (known after apply)
              + get_password_data                    = false
              + host_id                              = (known after apply)
              + host_resource_group_arn              = (known after apply)
              + iam_instance_profile                 = (known after apply)
              + id                                   = (known after apply)
              + instance_initiated_shutdown_behavior = (known after apply)
              + instance_lifecycle                   = (known after apply)
              + instance_state                       = (known after apply)
              + instance_type                        = "t2.micro"
              + ipv6_address_count                   = (known after apply)
              + ipv6_addresses                       = (known after apply)
              + key_name                             = (known after apply)
              + monitoring                           = (known after apply)
              + outpost_arn                          = (known after apply)
              + password_data                        = (known after apply)
              + placement_group                      = (known after apply)
              + placement_partition_number           = (known after apply)
              + primary_network_interface_id         = (known after apply)
              + private_dns                          = (known after apply)
              + private_ip                           = (known after apply)
              + public_dns                           = (known after apply)
              + public_ip                            = (known after apply)
              + secondary_private_ips                = (known after apply)
              + security_groups                      = (known after apply)
              + source_dest_check                    = true
              + spot_instance_request_id             = (known after apply)
              + subnet_id                            = (known after apply)
              + tags                                 = {
                  + "Name" = "TerraWeek-Server"
                }
              + tags_all                             = {
                  + "Name" = "TerraWeek-Server"
                }
              + tenancy                              = (known after apply)
              + user_data                            = (known after apply)
              + user_data_base64                     = (known after apply)
              + user_data_replace_on_change          = false
              + vpc_security_group_ids               = (known after apply)
        
              + capacity_reservation_specification (known after apply)
        
              + cpu_options (known after apply)
        
              + ebs_block_device (known after apply)
        
              + enclave_options (known after apply)
        
              + ephemeral_block_device (known after apply)
        
              + instance_market_options (known after apply)
        
              + maintenance_options (known after apply)
        
              + metadata_options (known after apply)
        
              + network_interface (known after apply)
        
              + private_dns_name_options (known after apply)
        
              + root_block_device (known after apply)
            }
        
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
        
          # aws_s3_bucket.app_logs will be created
          + resource "aws_s3_bucket" "app_logs" {
              + acceleration_status         = (known after apply)
              + acl                         = (known after apply)
              + arn                         = (known after apply)
              + bucket                      = "chetan-terraweek-app-logs-2026"
              + bucket_domain_name          = (known after apply)
              + bucket_prefix               = (known after apply)
              + bucket_regional_domain_name = (known after apply)
              + force_destroy               = false
              + hosted_zone_id              = (known after apply)
              + id                          = (known after apply)
              + object_lock_enabled         = (known after apply)
              + policy                      = (known after apply)
              + region                      = (known after apply)
              + request_payer               = (known after apply)
              + tags                        = {
                  + "Name" = "TerraWeek-App-Logs"
                }
              + tags_all                    = {
                  + "Name" = "TerraWeek-App-Logs"
                }
              + website_domain              = (known after apply)
              + website_endpoint            = (known after apply)
        
              + cors_rule (known after apply)
        
              + grant (known after apply)
        
              + lifecycle_rule (known after apply)
        
              + logging (known after apply)
        
              + object_lock_configuration (known after apply)
        
              + replication_configuration (known after apply)
        
              + server_side_encryption_configuration (known after apply)
        
              + versioning (known after apply)
        
              + website (known after apply)
            }
        
          # aws_security_group.main will be created
          + resource "aws_security_group" "main" {
              + arn                    = (known after apply)
              + description            = "Allow SSH and HTTP traffic"
              + egress                 = [
                  + {
                      + cidr_blocks      = [
                          + "0.0.0.0/0",
                        ]
                      + from_port        = 0
                      + ipv6_cidr_blocks = []
                      + prefix_list_ids  = []
                      + protocol         = "-1"
                      + security_groups  = []
                      + self             = false
                      + to_port          = 0
                        # (1 unchanged attribute hidden)
                    },
                ]
              + id                     = (known after apply)
              + ingress                = [
                  + {
                      + cidr_blocks      = [
                          + "0.0.0.0/0",
                        ]
                      + description      = "HTTP"
                      + from_port        = 80
                      + ipv6_cidr_blocks = []
                      + prefix_list_ids  = []
                      + protocol         = "tcp"
                      + security_groups  = []
                      + self             = false
                      + to_port          = 80
                    },
                  + {
                      + cidr_blocks      = [
                          + "0.0.0.0/0",
                        ]
                      + description      = "SSH"
                      + from_port        = 22
                      + ipv6_cidr_blocks = []
                      + prefix_list_ids  = []
                      + protocol         = "tcp"
                      + security_groups  = []
                      + self             = false
                      + to_port          = 22
                    },
                ]
              + name                   = "TerraWeek-SG"
              + name_prefix            = (known after apply)
              + owner_id               = (known after apply)
              + revoke_rules_on_delete = false
              + tags                   = {
                  + "Name" = "TerraWeek-SG"
                }
              + tags_all               = {
                  + "Name" = "TerraWeek-SG"
                }
              + vpc_id                 = (known after apply)
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
        
        Plan: 8 to add, 0 to change, 0 to destroy.

Now visualize the entire dependency tree:

        terraform graph | dot -Tpng > graph.png
        
If you don't have `dot` (Graphviz) installed, use:

        terraform graph

- terraform graph

         digraph G {
          rankdir = "RL";
          node [shape = rect, fontname = "sans-serif"];
          "data.aws_ami.amazon_linux_2" [label="data.aws_ami.amazon_linux_2"];
          "aws_instance.main" [label="aws_instance.main"];
          "aws_internet_gateway.gw" [label="aws_internet_gateway.gw"];
          "aws_route_table.example" [label="aws_route_table.example"];
          "aws_route_table_association.example" [label="aws_route_table_association.example"];
          "aws_s3_bucket.app_logs" [label="aws_s3_bucket.app_logs"];
          "aws_security_group.main" [label="aws_security_group.main"];
          "aws_subnet.main" [label="aws_subnet.main"];
          "aws_vpc.main" [label="aws_vpc.main"];
          "aws_instance.main" -> "data.aws_ami.amazon_linux_2";
          "aws_instance.main" -> "aws_security_group.main";
          "aws_instance.main" -> "aws_subnet.main";
          "aws_internet_gateway.gw" -> "aws_vpc.main";
          "aws_route_table.example" -> "aws_internet_gateway.gw";
          "aws_route_table_association.example" -> "aws_route_table.example";
          "aws_route_table_association.example" -> "aws_subnet.main";
          "aws_s3_bucket.app_logs" -> "aws_instance.main";
          "aws_security_group.main" -> "aws_vpc.main";
          "aws_subnet.main" -> "aws_vpc.main";
        }

and paste the output into an online Graphviz viewer.

Document: When would you use `depends_on` in real projects? Give two examples.

        Use depends_on when there is a real dependency between resources that Terraform cannot determine from normal resource references.

- Example 1 — IAM permission before an application
- Example 2 — Application infrastructure after foundational infrastructure

---

Task 6: Lifecycle Rules and Destroy

1) Add a `lifecycle` block to your EC2 instance:

        lifecycle {
          create_before_destroy = true
        }

2) Change the AMI ID to a different one and run `terraform plan` -- observe that Terraform plans to create the new instance before destroying the old one

        -/+ aws_instance.main

3) Destroy everything:

        terraform destroy

        Plan: 0 to add, 0 to change, 8 to destroy.

4) Watch the destroy order -- Terraform destroys in reverse dependency order. Verify in the AWS console that everything is cleaned up.



Document: What are the three lifecycle arguments (`create_before_destroy`, `prevent_destroy`, `ignore_changes`) and when would you use each?

| Argument              | What it does                                       | Typical use                   |
|-----------------------|----------------------------------------------------|-------------------------------|
| create_before_destroy | Creates replacement before destroying old resource | Minimize downtime             |
| prevent_destroy       | Prevents Terraform from destroying the resource    | Protect critical resources    |
| ignore_changes        | Ignores changes to specified attributes            | Allow external/manual changes |


`providers.tf`
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-west-2"
}
```

`main.tf`
```hcl
        data "aws_ami" "amazon_linux_2" {
          most_recent = true
        
          owners = ["amazon"]
        
          filter {
            name   = "name"
            values = ["amzn2-ami-hvm-*-x86_64-gp2"]
          }
        
          filter {
            name   = "state"
            values = ["available"]
          }
        }
        
        resource "aws_vpc" "main" {
          cidr_block = "10.0.0.0/16"
          tags = {
            Name = "TerraWeek-VPC"
          }
        }
        
        resource "aws_subnet" "main" {
          vpc_id                  = aws_vpc.main.id
          cidr_block              = "10.0.1.0/24"
          map_public_ip_on_launch = true
        
          tags = {
            Name = "TerraWeek-Public-Subnet"
          }
        }
        
        resource "aws_internet_gateway" "gw" {
          vpc_id = aws_vpc.main.id
        
          tags = {
            Name = "main"
          }
        }
        
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
        
        resource "aws_route_table_association" "example" {
          subnet_id      = aws_subnet.main.id
          route_table_id = aws_route_table.example.id
        }
        
        resource "aws_security_group" "main" {
          name        = "TerraWeek-SG"
          description = "Allow SSH and HTTP traffic"
          vpc_id      = aws_vpc.main.id
        
          ingress {
            description = "SSH"
            from_port   = 22
            to_port     = 22
            protocol    = "tcp"
            cidr_blocks = ["0.0.0.0/0"]
          }
        
          ingress {
            description = "HTTP"
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
            Name = "TerraWeek-SG"
          }
        }
        
        resource "aws_instance" "main" {
          ami                         = data.aws_ami.amazon_linux_2.id
          instance_type               = "t2.micro"
          subnet_id                   = aws_subnet.main.id
          vpc_security_group_ids      = [aws_security_group.main.id]
          associate_public_ip_address = true
        
          lifecycle {
            create_before_destroy = true
          }
              
          tags = {
            Name = "TerraWeek-Server"
          }
        }
        
        resource "aws_s3_bucket" "app_logs" {
          bucket = "chetan-terraweek-app-logs-2026"
          depends_on = [aws_instance.main]
        
          tags = {
            Name = "TerraWeek-App-Logs"
          }
        }
```
