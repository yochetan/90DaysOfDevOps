Task 1: Understand Infrastructure as Code

Before touching the terminal, research and write short notes on:

1) What is Infrastructure as Code (IaC)? Why does it matter in DevOps?

        IaC allows me to treat infrastructure like application code — I can write it, review it, version it, change it, and reproduce it.

2) What problems does IaC solve compared to manually creating resources in the AWS console?

        Manually creating infrastructure through the AWS Console can work for small experiments, but it becomes difficult to manage as the environment grows.

        IaC helps solve problems such as:

        - Human errors: Manual configuration can lead to accidentally choosing the wrong settings or forgetting a resource.
        
        - Inconsistent environments: Development and production environments can end up configured differently.
        
        - Repetition: Creating the same resources again and again manually takes time.
        
        - Lack of history: It's difficult to know exactly who changed a console setting and what the previous configuration was.
        
        - Poor scalability: Managing dozens or hundreds of resources manually becomes impractical.
        
        - Difficult recovery: Rebuilding an environment after accidental deletion is much harder without documented configuration.
        
        With IaC, the infrastructure configuration can be stored in Git, reviewed through pull requests, and automatically applied through CI/CD pipelines.

3) How is Terraform different from AWS CloudFormation, Ansible, and Pulumi?

| Tool                   | Main Difference                                                                                                                                                                     | 
|------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Terraform**          | General-purpose IaC tool that can manage infrastructure across many cloud providers and services. Uses **HCL** and maintains a state file.                                          |
| **AWS CloudFormation** | AWSs native IaC service. It is mainly designed for managing AWS resources and integrates deeply with AWS.                                                                           |
| **Ansible**            | Primarily an automation and configuration-management tool. It is commonly used to configure existing servers, install packages, deploy applications, and perform operational tasks. |
| **Pulumi**             | Similar to Terraform in infrastructure management, but allows infrastructure to be defined using general-purpose programming languages such as Python, TypeScript, Go, and C#.      |

4) What does it mean that Terraform is "declarative" and "cloud-agnostic"?

- Declarative

        Terraform is declarative, meaning I describe what I want the final infrastructure to look like, rather than writing every step required to create it.

- Cloud-agnostic

        Cloud-agnostic means Terraform isn't limited to a single cloud provider.

Write this in your own words -- not copy-pasted definitions.

---

Task 2: Install Terraform and Configure AWS

1) Install Terraform:

        # macOS
        brew tap hashicorp/tap
        brew install hashicorp/tap/terraform
        
        # Linux (amd64)
        wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
        echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
        sudo apt update && sudo apt install terraform
        
        # Windows
        choco install terraform

* I did using linux as I am doing it in an AWS EC2 instance

2) Verify:

- terraform -version
        
        Terraform v1.16.2
        on linux_amd64

3) Install and configure the AWS CLI:

        aws configure
        # Enter your Access Key ID, Secret Access Key, default region (e.g., ap-south-1), output format (json)

4) Verify AWS access:

- aws sts get-caller-identity
        
        {
            "UserId": "AIDAY4Z4RYDCWKQYRC42V",
            "Account": "611622961349",
            "Arn": "arn:aws:iam::611622961349:user/chota-chetan"
        }

You should see your AWS account ID and ARN.

---

Task 3: Your First Terraform Config -- Create an S3 Bucket

Create a project directory and write your first Terraform config:

        mkdir terraform-basics && cd terraform-basics

Create a file called `main.tf` with:

1) A `terraform` block with `required_providers` specifying the `aws` provider

        terraform {
          required_providers {
            aws = {
              source  = "hashicorp/aws"
            }
          }
        }

2) A `provider "aws"` block with your region

        provider "aws" {
          region = "us-west-2"
        }

3) A `resource "aws_s3_bucket"` that creates a bucket with a globally unique name

        resource "aws_s3_bucket" "terraform_bucket" {
          bucket = "chetan-terraform-bucket"
        }

Run the Terraform lifecycle:

- terraform init      # Download the AWS provider

        Initializing the backend...
        
        Initializing provider plugins...
        - Finding latest version of hashicorp/aws...
        - Installing hashicorp/aws v6.64.0...
        - Installed hashicorp/aws v6.64.0 (signed by HashiCorp)
        
        Terraform has created a lock file .terraform.lock.hcl to record the provider
        selections it made above. Include this file in your version control repository
        so that Terraform can guarantee to make the same selections by default when
        you run "terraform init" in the future.
        
        Terraform has been successfully initialized!

- terraform plan      # Preview what will be created

        Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with
        the following symbols:
          + create
        
        Terraform will perform the following actions:
        
          # aws_s3_bucket.terraform_bucket will be created
          + resource "aws_s3_bucket" "terraform_bucket" {
              + acceleration_status         = (known after apply)
              + acl                         = (known after apply)
              + arn                         = (known after apply)
              + bucket                      = "chetan-terraform-bucket"
              + bucket_domain_name          = (known after apply)
              + bucket_namespace            = (known after apply)
              + bucket_prefix               = (known after apply)
              + bucket_region               = (known after apply)
              + bucket_regional_domain_name = (known after apply)
              + force_destroy               = false
              + hosted_zone_id              = (known after apply)
              + id                          = (known after apply)
              + object_lock_enabled         = (known after apply)
              + policy                      = (known after apply)
              + region                      = "us-west-2"
              + request_payer               = (known after apply)
              + tags_all                    = (known after apply)
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
        
        Plan: 1 to add, 0 to change, 0 to destroy.


- terraform apply     # Create the bucket (type 'yes' to confirm)

        Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with
        the following symbols:
          + create
        
        Terraform will perform the following actions:
        
          # aws_s3_bucket.terraform_bucket will be created
          + resource "aws_s3_bucket" "terraform_bucket" {
              + acceleration_status         = (known after apply)
              + acl                         = (known after apply)
              + arn                         = (known after apply)
              + bucket                      = "chetan-terraform-bucket"
              + bucket_domain_name          = (known after apply)
              + bucket_namespace            = (known after apply)
              + bucket_prefix               = (known after apply)
              + bucket_region               = (known after apply)
              + bucket_regional_domain_name = (known after apply)
              + force_destroy               = false
              + hosted_zone_id              = (known after apply)
              + id                          = (known after apply)
              + object_lock_enabled         = (known after apply)
              + policy                      = (known after apply)
              + region                      = "us-west-2"
              + request_payer               = (known after apply)
              + tags_all                    = (known after apply)
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
        
        Plan: 1 to add, 0 to change, 0 to destroy.
        
        Do you want to perform these actions?
          Terraform will perform the actions described above.
          Only 'yes' will be accepted to approve.
        
          Enter a value: yes
        
        aws_s3_bucket.terraform_bucket: Creating...
        aws_s3_bucket.terraform_bucket: Creation complete after 0s [id=chetan-terraform-bucket]
        
        Apply complete! Resources: 1 added, 0 changed, 0 destroyed.


Go to the AWS S3 console and verify your bucket exists.

- aws s3 ls

        2026-09-14 21:09:47 chetan-terraform-bucket

Document: What did `terraform init` download? What does the `.terraform/` directory contain?

- terraform init

        Terraform downloaded the AWS provider plugin from the Terraform Registry.

- .terraform/

        The .terraform/ directory contains Terraform's local working data, including downloaded provider plugins and related dependency information.

---

Task 4: Add an EC2 Instance

In the same `main.tf`, add:

1) A `resource "aws_instance"` using AMI `ami-0f5ee92e2d63afc18` (Amazon Linux 2 in ap-south-1 -- use the correct AMI for your region)

        provider "aws" {
          region = "us-west-2"
        }
        resource "aws_instance" "day1" {
          ami           = "ami-02167eae61967e403"
          instance_type = "t3.micro"
        
          tags = {
            Name = "TerraWeek-Day1"
          }
        }

2) Set instance type to `t3.micro`

        instance_type = "t3.micro"

3) Add a tag: `Name = "TerraWeek-Day1"`

        tags = {
            Name = "TerraWeek-Day1"
          }

Run:

- terraform plan      # You should see 1 resource to add (bucket already exists)

        aws_s3_bucket.terraform_bucket: Refreshing state... [id=chetan-terraform-bucket]
        
        Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with
        the following symbols:
          + create
        
        Terraform will perform the following actions:
        
          # aws_instance.day1 will be created
          + resource "aws_instance" "day1" {
              + ami                                  = "ami-02167eae61967e403"
              + arn                                  = (known after apply)
              + associate_public_ip_address          = (known after apply)
              + availability_zone                    = (known after apply)
              + disable_api_stop                     = (known after apply)
              + disable_api_termination              = (known after apply)
              + ebs_optimized                        = (known after apply)
              + enable_primary_ipv6                  = (known after apply)
              + force_destroy                        = false
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
              + placement_group_id                   = (known after apply)
              + placement_partition_number           = (known after apply)
              + primary_network_interface_id         = (known after apply)
              + private_dns                          = (known after apply)
              + private_ip                           = (known after apply)
              + public_dns                           = (known after apply)
              + public_ip                            = (known after apply)
              + region                               = "us-west-2"
              + secondary_private_ips                = (known after apply)
              + security_groups                      = (known after apply)
              + source_dest_check                    = true
              + spot_instance_request_id             = (known after apply)
              + subnet_id                            = (known after apply)
              + tags                                 = {
                  + "Name" = "TerraWeek-Day1"
                }
              + tags_all                             = {
                  + "Name" = "TerraWeek-Day1"
                }
              + tenancy                              = (known after apply)
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
        
              + primary_network_interface (known after apply)
        
              + private_dns_name_options (known after apply)
        
              + root_block_device (known after apply)
        
              + secondary_network_interface (known after apply)
            }
        
        Plan: 1 to add, 0 to change, 0 to destroy.

- terraform apply

        Terraform will perform the following actions:
        
          # aws_instance.day1 will be created
          + resource "aws_instance" "day1" {
              + ami                                  = "ami-02167eae61967e403"
              + arn                                  = (known after apply)
              + associate_public_ip_address          = (known after apply)
              + availability_zone                    = (known after apply)
              + disable_api_stop                     = (known after apply)
              + disable_api_termination              = (known after apply)
              + ebs_optimized                        = (known after apply)
              + enable_primary_ipv6                  = (known after apply)
              + force_destroy                        = false
              + get_password_data                    = false
              + host_id                              = (known after apply)
              + host_resource_group_arn              = (known after apply)
              + iam_instance_profile                 = (known after apply)
              + id                                   = (known after apply)
              + instance_initiated_shutdown_behavior = (known after apply)
              + instance_lifecycle                   = (known after apply)
              + instance_state                       = (known after apply)
              + instance_type                        = "t3.micro"
              + ipv6_address_count                   = (known after apply)
              + ipv6_addresses                       = (known after apply)
              + key_name                             = (known after apply)
              + monitoring                           = (known after apply)
              + outpost_arn                          = (known after apply)
              + password_data                        = (known after apply)
              + placement_group                      = (known after apply)
              + placement_group_id                   = (known after apply)
              + placement_partition_number           = (known after apply)
              + primary_network_interface_id         = (known after apply)
              + private_dns                          = (known after apply)
              + private_ip                           = (known after apply)
              + public_dns                           = (known after apply)
              + public_ip                            = (known after apply)
              + region                               = "us-west-2"
              + secondary_private_ips                = (known after apply)
              + security_groups                      = (known after apply)
              + source_dest_check                    = true
              + spot_instance_request_id             = (known after apply)
              + subnet_id                            = (known after apply)
              + tags                                 = {
                  + "Name" = "TerraWeek-Day1"
                }
              + tags_all                             = {
                  + "Name" = "TerraWeek-Day1"
                }
              + tenancy                              = (known after apply)
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
        
              + primary_network_interface (known after apply)
        
              + private_dns_name_options (known after apply)
        
              + root_block_device (known after apply)
        
              + secondary_network_interface (known after apply)
            }
        
        Plan: 1 to add, 0 to change, 0 to destroy.
        
        Do you want to perform these actions?
          Terraform will perform the actions described above.
          Only 'yes' will be accepted to approve.
        
          Enter a value: yes
        
        aws_instance.day1: Creating...
        aws_instance.day1: Still creating... [00m10s elapsed]
        aws_instance.day1: Creation complete after 13s [id=i-0baf6efe6a45e73f4]
        
        Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Go to the AWS EC2 console and verify your instance is running with the correct name tag.

        Yes it is running with the correct name tag.

Document: How does Terraform know the S3 bucket already exists and only the EC2 instance needs to be created?

        Terraform knows the S3 bucket already exists because it maintains a state file (terraform.tfstate) containing information about resources it has previously created and manages. During terraform plan, Terraform compares the configuration with the state and the actual infrastructure. Since the S3 bucket is already tracked and matches the configuration, Terraform leaves it unchanged and plans only the new EC2 instance for creation.

---

Task 5: Understand the State File

Terraform tracks everything it creates in a state file. Time to inspect it.

1) Open `terraform.tfstate` in your editor -- read the JSON structure



2) Run these commands and document what each returns:

- terraform show                          # Human-readable view of current state

        # aws_instance.day1:
        resource "aws_instance" "day1" {
            ami                                  = "ami-02167eae61967e403"
            arn                                  = "arn:aws:ec2:us-west-2:611622961349:instance/i-0baf6efe6a45e73f4"
            associate_public_ip_address          = true
            availability_zone                    = "us-west-2b"
            disable_api_stop                     = false
            disable_api_termination              = false
            ebs_optimized                        = false
            force_destroy                        = false
            get_password_data                    = false
            hibernation                          = false
            host_id                              = null
            iam_instance_profile                 = null
            id                                   = "i-0baf6efe6a45e73f4"
            instance_initiated_shutdown_behavior = "stop"
            instance_lifecycle                   = null
            instance_state                       = "running"
            instance_type                        = "t3.micro"
            ipv6_address_count                   = 0
            ipv6_addresses                       = []
            key_name                             = null
            monitoring                           = false
            outpost_arn                          = null
            password_data                        = null
            placement_group                      = null
            placement_group_id                   = null
            placement_partition_number           = 0
            primary_network_interface_id         = "eni-003e20cd0ee15473c"
            private_dns                          = "ip-172-31-27-19.us-west-2.compute.internal"
            private_ip                           = "172.31.27.19"
            public_dns                           = "ec2-54-203-196-30.us-west-2.compute.amazonaws.com"
            public_ip                            = "54.203.196.30"
            region                               = "us-west-2"
            secondary_private_ips                = []
            security_groups                      = [
                "default",
            ]
            source_dest_check                    = true
            spot_instance_request_id             = null
            subnet_id                            = "subnet-0ad91aa7f29ac07ad"
            tags                                 = {
                "Name" = "TerraWeek-Day1"
            }
            tags_all                             = {
                "Name" = "TerraWeek-Day1"
            }
            tenancy                              = "default"
            user_data_replace_on_change          = false
            vpc_security_group_ids               = [
                "sg-08065dcd0f7e73b10",
            ]
        
            capacity_reservation_specification {
                capacity_reservation_preference = "open"
            }
        
            cpu_options {
                amd_sev_snp           = null
                core_count            = 1
                nested_virtualization = null
                threads_per_core      = 2
            }
        
            credit_specification {
                cpu_credits = "unlimited"
            }
        
            enclave_options {
                enabled = false
            }
        
            maintenance_options {
                auto_recovery = "default"
            }
        
            metadata_options {
                http_endpoint               = "enabled"
                http_protocol_ipv6          = "disabled"
                http_put_response_hop_limit = 2
                http_tokens                 = "required"
                instance_metadata_tags      = "disabled"
            }
        
            primary_network_interface {
                delete_on_termination = true
                network_interface_id  = "eni-003e20cd0ee15473c"
            }
        
            private_dns_name_options {
                enable_resource_name_dns_a_record    = false
                enable_resource_name_dns_aaaa_record = false
                hostname_type                        = "ip-name"
            }
        
            root_block_device {
                delete_on_termination = true
                device_name           = "/dev/sda1"
                encrypted             = false
                iops                  = 3000
                kms_key_id            = null
                tags                  = {}
                tags_all              = {}
                throughput            = 125
                volume_id             = "vol-04db3ad83302f7296"
                volume_size           = 8
                volume_type           = "gp3"
            }
        }
        
        # aws_s3_bucket.terraform_bucket:
        resource "aws_s3_bucket" "terraform_bucket" {
            acceleration_status         = null
            arn                         = "arn:aws:s3:::chetan-terraform-bucket"
            bucket                      = "chetan-terraform-bucket"
            bucket_domain_name          = "chetan-terraform-bucket.s3.amazonaws.com"
            bucket_namespace            = "global"
            bucket_prefix               = null
            bucket_region               = "us-west-2"
            bucket_regional_domain_name = "chetan-terraform-bucket.s3.us-west-2.amazonaws.com"
            force_destroy               = false
            hosted_zone_id              = "Z3BJ6K6RIION7M"
            id                          = "chetan-terraform-bucket"
            object_lock_enabled         = false
            policy                      = null
            region                      = "us-west-2"
            request_payer               = "BucketOwner"
            tags                        = {}
            tags_all                    = {}
        
            grant {
                id          = "6143a96047b515dbb3a4d819e7f702c1878a4c2d66b923cb848ec1a8f8683217"
                permissions = [
                    "FULL_CONTROL",
                ]
                type        = "CanonicalUser"
                uri         = null
            }
        
            server_side_encryption_configuration {
                rule {
                    bucket_key_enabled = false
        
                    apply_server_side_encryption_by_default {
                        kms_master_key_id = null
                        sse_algorithm     = "AES256"
                    }
                }
            }
        
            versioning {
                enabled    = false
                mfa_delete = false
            }
        }

- terraform state list                    # List all resources Terraform manages
        
        aws_instance.day1
        aws_s3_bucket.terraform_bucket

- terraform state show aws_s3_bucket.<name>   # Detailed view of a specific resource

        # aws_s3_bucket.terraform_bucket:
        resource "aws_s3_bucket" "terraform_bucket" {
            acceleration_status         = null
            arn                         = "arn:aws:s3:::chetan-terraform-bucket"
            bucket                      = "chetan-terraform-bucket"
            bucket_domain_name          = "chetan-terraform-bucket.s3.amazonaws.com"
            bucket_namespace            = "global"
            bucket_prefix               = null
            bucket_region               = "us-west-2"
            bucket_regional_domain_name = "chetan-terraform-bucket.s3.us-west-2.amazonaws.com"
            force_destroy               = false
            hosted_zone_id              = "Z3BJ6K6RIION7M"
            id                          = "chetan-terraform-bucket"
            object_lock_enabled         = false
            policy                      = null
            region                      = "us-west-2"
            request_payer               = "BucketOwner"
            tags                        = {}
            tags_all                    = {}
        
            grant {
                id          = "6143a96047b515dbb3a4d819e7f702c1878a4c2d66b923cb848ec1a8f8683217"
                permissions = [
                    "FULL_CONTROL",
                ]
                type        = "CanonicalUser"
                uri         = null
            }
        
            server_side_encryption_configuration {
                rule {
                    bucket_key_enabled = false
        
                    apply_server_side_encryption_by_default {
                        kms_master_key_id = null
                        sse_algorithm     = "AES256"
                    }
                }
            }
        
            versioning {
                enabled    = false
                mfa_delete = false
            }
        }

- terraform state show aws_instance.<name>

        # aws_instance.day1:
        resource "aws_instance" "day1" {
            ami                                  = "ami-02167eae61967e403"
            arn                                  = "arn:aws:ec2:us-west-2:611622961349:instance/i-0baf6efe6a45e73f4"
            associate_public_ip_address          = true
            availability_zone                    = "us-west-2b"
            disable_api_stop                     = false
            disable_api_termination              = false
            ebs_optimized                        = false
            force_destroy                        = false
            get_password_data                    = false
            hibernation                          = false
            host_id                              = null
            iam_instance_profile                 = null
            id                                   = "i-0baf6efe6a45e73f4"
            instance_initiated_shutdown_behavior = "stop"
            instance_lifecycle                   = null
            instance_state                       = "running"
            instance_type                        = "t3.micro"
            ipv6_address_count                   = 0
            ipv6_addresses                       = []
            key_name                             = null
            monitoring                           = false
            outpost_arn                          = null
            password_data                        = null
            placement_group                      = null
            placement_group_id                   = null
            placement_partition_number           = 0
            primary_network_interface_id         = "eni-003e20cd0ee15473c"
            private_dns                          = "ip-172-31-27-19.us-west-2.compute.internal"
            private_ip                           = "172.31.27.19"
            public_dns                           = "ec2-54-203-196-30.us-west-2.compute.amazonaws.com"
            public_ip                            = "54.203.196.30"
            region                               = "us-west-2"
            secondary_private_ips                = []
            security_groups                      = [
                "default",
            ]
            source_dest_check                    = true
            spot_instance_request_id             = null
            subnet_id                            = "subnet-0ad91aa7f29ac07ad"
            tags                                 = {
                "Name" = "TerraWeek-Day1"
            }
            tags_all                             = {
                "Name" = "TerraWeek-Day1"
            }
            tenancy                              = "default"
            user_data_replace_on_change          = false
            vpc_security_group_ids               = [
                "sg-08065dcd0f7e73b10",
            ]
        
            capacity_reservation_specification {
                capacity_reservation_preference = "open"
            }
        
            cpu_options {
                amd_sev_snp           = null
                core_count            = 1
                nested_virtualization = null
                threads_per_core      = 2
            }
        
            credit_specification {
                cpu_credits = "unlimited"
            }
        
            enclave_options {
                enabled = false
            }
        
            maintenance_options {
                auto_recovery = "default"
            }
        
            metadata_options {
                http_endpoint               = "enabled"
                http_protocol_ipv6          = "disabled"
                http_put_response_hop_limit = 2
                http_tokens                 = "required"
                instance_metadata_tags      = "disabled"
            }
        
            primary_network_interface {
                delete_on_termination = true
                network_interface_id  = "eni-003e20cd0ee15473c"
            }
        
            private_dns_name_options {
                enable_resource_name_dns_a_record    = false
                enable_resource_name_dns_aaaa_record = false
                hostname_type                        = "ip-name"
            }
        
            root_block_device {
                delete_on_termination = true
                device_name           = "/dev/sda1"
                encrypted             = false
                iops                  = 3000
                kms_key_id            = null
                tags                  = {}
                tags_all              = {}
                throughput            = 125
                volume_id             = "vol-04db3ad83302f7296"
                volume_size           = 8
                volume_type           = "gp3"
            }
        }


3) Answer these questions in your notes:

* What information does the state file store about each resource?

        The Terraform state file (terraform.tfstate) stores information that Terraform uses to track and manage the infrastructure it created.

        For each resource, it can contain:

        - Resource type and name — e.g. aws_instance.web
        - Resource ID — the actual AWS resource ID, such as an EC2 instance ID
        - Resource attributes — configuration and values returned by the provider, such as AMI ID, instance type, subnet, IP address, tags, etc.
        - Dependencies — relationships between resources
        - Provider information — which provider manages the resource
        - Terraform metadata — information Terraform needs to compare the desired configuration with the current infrastructure

* Why should you never manually edit the state file?

        Terraform state is managed by Terraform, and its structure can be complex.

        Manually editing it can:

        - Corrupt the state
        - Cause Terraform to lose track of resources
        - Create inconsistencies between Terraform and AWS
        - Cause Terraform to incorrectly create, modify, or destroy resources
        - Break resource dependencies

* Why should the state file not be committed to Git?

        The state file may contain sensitive information, such as:
        
        - Passwords
        - Database credentials
        - Access-related information
        - Private IP addresses
        - Resource IDs
        - Other provider-returned sensitive values

* State file: Tracks Terraform-managed resources and their current attributes.

* Don't manually edit: It can corrupt Terraform's understanding of your infrastructure.

* Don't commit to Git: It may contain sensitive data and can cause conflicts; use a secure remote backend instead.

---

Task 6: Modify, Plan, and Destroy

1) Change the EC2 instance tag from `"TerraWeek-Day1"` to `"TerraWeek-Modified"` in your main.tf

         tags = {
            Name = "TerraWeek-Modified"
          }

2) Run terraform plan and read the output carefully:

- What do the ~, +, and - symbols mean?

| Symbol | Meaning             | What happens                             |
| ------ | ------------------- | -----------------------------------------|
| `+`    | **Create**          | Resource will be created                 |
| `-`    | **Destroy**         | Resource will be deleted                 |
| `~`    | **Update in-place** | Existing resource will be modified       |
| `-/+`  | **Replace**         | Resource will be destroyed and recreated |

- Is this an in-place update or a destroy-and-recreate?

        Update in-place

3) Apply the change

        terraform apply

4) Verify the tag changed in the AWS console

        Yeah, the current tag is TerraWeek-Modified

5) Finally, destroy everything:

        terraform destroy

6) Verify in the AWS console -- both the S3 bucket and EC2 instance should be gone

        yes both are destroyed/deleted and gone from AWS console
