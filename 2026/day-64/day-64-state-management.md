Task 1: Inspect Your Current State

Use your Day 63 config (or create a small config with a VPC and EC2 instance). Apply it and then explore the state:

        terraform show                                    # Full state in human-readable format        
        terraform state list                              # All resources tracked by Terraform
        terraform state show aws_instance.<name>          # Every attribute of the instance
        terraform state show aws_vpc.<name>               # Every attribute of the VPC

Answer:

1) How many resources does Terraform track?

        data.aws_ami.amazon_linux
        data.aws_availability_zones.available
        aws_instance.main
        aws_internet_gateway.gw
        aws_route_table.example
        aws_route_table_association.example
        aws_s3_bucket.app_logs
        aws_security_group.main
        aws_subnet.main
        aws_vpc.main
        
        10 resources.

2) What attributes does the state store for an EC2 instance? (hint: way more than what you defined)

        # aws_instance.main:
        resource "aws_instance" "main" {
            ami                                  = "ami-01477f93b365aa11a"
            arn                                  = "arn:aws:ec2:us-west-2:611622961349:instance/i-0d4dd5395e66079c2"
            associate_public_ip_address          = true
            availability_zone                    = "us-west-2a"
            cpu_core_count                       = 1
            cpu_threads_per_core                 = 2
            disable_api_stop                     = false
            disable_api_termination              = false
            ebs_optimized                        = false
            get_password_data                    = false
            hibernation                          = false
            host_id                              = null
            iam_instance_profile                 = null
            id                                   = "i-0d4dd5395e66079c2"
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
            placement_partition_number           = 0
            primary_network_interface_id         = "eni-098ec05f77bca00cd"
            private_dns                          = "ip-10-0-1-180.us-west-2.compute.internal"
            private_ip                           = "10.0.1.180"
            public_dns                           = null
            public_ip                            = "34.213.90.43"
            secondary_private_ips                = []
            security_groups                      = []
            source_dest_check                    = true
            spot_instance_request_id             = null
            subnet_id                            = "subnet-0f4bbe093366313a8"
            tags                                 = {
                "Environment" = "dev"
                "ManagedBy"   = "Terraform"
                "Name"        = "terraweek-dev-server"
                "Project"     = "terraweek"
            }
            tags_all                             = {
                "Environment" = "dev"
                "ManagedBy"   = "Terraform"
                "Name"        = "terraweek-dev-server"
                "Project"     = "terraweek"
            }
            tenancy                              = "default"
            user_data_replace_on_change          = false
            vpc_security_group_ids               = [
                "sg-0fa673d9571191196",
            ]
        
            capacity_reservation_specification {
                capacity_reservation_preference = "open"
            }
        
            cpu_options {
                amd_sev_snp      = null
                core_count       = 1
                threads_per_core = 2
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
                http_put_response_hop_limit = 1
                http_tokens                 = "optional"
                instance_metadata_tags      = "disabled"
            }
        
            private_dns_name_options {
                enable_resource_name_dns_a_record    = false
                enable_resource_name_dns_aaaa_record = false
                hostname_type                        = "ip-name"
            }
        
            root_block_device {
                delete_on_termination = true
                device_name           = "/dev/xvda"
                encrypted             = false
                iops                  = 100
                kms_key_id            = null
                tags                  = {}
                tags_all              = {}
                throughput            = 0
                volume_id             = "vol-0e9e4b1053e6521f2"
                volume_size           = 8
                volume_type           = "gp2"
            }
        }

3) Open `terraform.tfstate` in an editor -- find the `serial` number. What does it represent?
        
        "serial": 30
        
        A monotonically increasing state revision/version number that changes when Terraform writes a new state.

---

Task 2: Set Up S3 Remote Backend

Storing state locally is dangerous -- one deleted file and you lose everything. Time to move it to S3.

1) First, create the backend infrastructure (do this manually or in a separate Terraform config):

        # Create S3 bucket for state storage
        aws s3api create-bucket \
          --bucket terraweek-state-<yourname> \
          --region ap-south-1 \
          --create-bucket-configuration LocationConstraint=ap-south-1
        
        # Enable versioning (so you can recover previous state)
        aws s3api put-bucket-versioning \
          --bucket terraweek-state-<yourname> \
          --versioning-configuration Status=Enabled
        
        # Create DynamoDB table for state locking
        aws dynamodb create-table \
          --table-name terraweek-state-lock \
          --attribute-definitions AttributeName=LockID,AttributeType=S \
          --key-schema AttributeName=LockID,KeyType=HASH \
          --billing-mode PAY_PER_REQUEST \
          --region ap-south-1

`backend-setup/main.tf`

        provider "aws" {
          region = "us-west-2"
        }
        
        resource "aws_s3_bucket" "terraform_state" {
          bucket = "terraweek-state-chetan"
        
          tags = {
            Name = "Terraform State"
          }
        }
        
        resource "aws_s3_bucket_versioning" "terraform_state" {
          bucket = aws_s3_bucket.terraform_state.id
        
          versioning_configuration {
            status = "Enabled"
          }
        }
        
        resource "aws_dynamodb_table" "terraform_lock" {
          name         = "terraweek-state-lock"
          billing_mode = "PAY_PER_REQUEST"
          hash_key     = "LockID"
        
          attribute {
            name = "LockID"
            type = "S"
          }
        }

2) Add the backend block to your Terraform config:

        terraform {
          backend "s3" {
            bucket         = "terraweek-state-<yourname>"
            key            = "dev/terraform.tfstate"
            region         = "ap-south-1"
            dynamodb_table = "terraweek-state-lock"
            encrypt        = true
          }
        }

3) Run:

        terraform init               

        Initializing the backend...
        Do you want to copy existing state to the new backend?
          Pre-existing state was found while migrating the previous "local" backend to the
          newly configured "s3" backend. No existing state was found in the newly
          configured "s3" backend. Do you want to copy this state to the new "s3"
          backend? Enter "yes" to copy and "no" to start with an empty state.
        
          Enter a value: yes
        
        Releasing state lock. This may take a few moments...
        
        Successfully configured the backend "s3"! Terraform will automatically
        use this backend unless the backend configuration changes.

Terraform will ask: "Do you want to copy existing state to the new backend?" -- say yes.

        yep.

4) Verify:

* Check the S3 bucket -- you should see `dev/terraform.tfstate`

- aws s3 ls s3://terraweek-state-chetan/dev/

        2026-10-08 04:26:30      24701 terraform.tfstate

* Your local `terraform.tfstate` should now be empty or gone

- dir terraform.tfstate


      Directory: C:\Users\gtmew\Desktop\Projects\terraform\terraform-aws-infra
        
        
        Mode                 LastWriteTime         Length Name                                                        
        ----                 -------------         ------ ----                                                        
        -a----         10/8/2026   4:26 AM              0 terraform.tfstate    

* Run `terraform plan` -- it should show no changes (state migrated correctly)

        No changes. Your infrastructure matches the configuration.

---

Task 3: Test State Locking

State locking prevents two people from running `terraform apply` at the same time and corrupting the state.

1) Open two terminals in the same project directory

        yes

2) In Terminal 1, run:

        terraform apply

        Plan: 8 to add, 0 to change, 0 to destroy.
        
        Changes to Outputs:
          + instance_id         = (known after apply)
          + instance_public_dns = (known after apply)
          + instance_public_ip  = (known after apply)
          + security_group_id   = (known after apply)
          + subnet_id           = (known after apply)
          + vpc_id              = (known after apply)
        
        Do you want to perform these actions?
          Terraform will perform the actions described above.
          Only 'yes' will be accepted to approve.
        
          Enter a value: 

3) While Terminal 1 is waiting for confirmation, in Terminal 2 run:

        terraform plan

        Error: Error acquiring the state lock
        │ 
        │ Error message: operation error DynamoDB: PutItem, https response error StatusCode: 400, RequestID:
        │ 7BNGBHEGJPP2HKAHNJMU5TU7QRVV4KQNSO5AEMVJF66Q9ASUAAJG, ConditionalCheckFailedException: The conditional
        │ request failed
        │ Lock Info:
        │   ID:        482a8f96-d2de-4e27-c6c1-59314598bed7
        │   Path:      terraweek-state-chetan/dev/terraform.tfstate
        │   Operation: OperationTypeApply
        │   Who:       CG\CG@CG
        │   Version:   1.16.3
        │   Created:   2026-10-07 23:06:44.597876 +0000 UTC
        │   Info:      
        │ 
        │ 
        │ Terraform acquires a state lock to protect the state from being written
        │ by multiple users at the same time. Please resolve the issue above and try
        │ again. For most commands, you can disable locking with the "-lock=false"
        │ flag, but this is not recommended.
        
4) Terminal 2 should show a lock error with a Lock ID

        Lock Info:
        │   ID:        482a8f96-d2de-4e27-c6c1-59314598bed7

*Document*: What is the error message? Why is locking critical for team environments?

5) After the test, if you get stuck with a stale lock:

        terraform force-unlock <LOCK_ID>

---

Task 4: Import an Existing Resource

Not everything starts with Terraform. Sometimes resources already exist in AWS and you need to bring them under Terraform management.

1) Manually create an S3 bucket in the AWS console -- name it `terraweek-import-test-<yourname>`

        created.

2) Write a `resource "aws_s3_bucket"` block in your config for this bucket (just the bucket name, nothing else)
        
        resource "aws_s3_bucket" "imported" {
          bucket = "terraweek-import-test-chetan"
        }

3) Import it:

        terraform import aws_s3_bucket.imported terraweek-import-test-<yourname>

        aws_s3_bucket.imported: Import prepared!
          Prepared aws_s3_bucket for import
        aws_s3_bucket.imported: Refreshing state... [id=terraweek-import-test-chetan]
        data.aws_availability_zones.available: Read complete after 1s [id=us-west-2]
        data.aws_ami.amazon_linux: Read complete after 2s [id=ami-01477f93b365aa11a]

        Import successful!
        
        The resources that were imported are shown above. These resources are now in
        your Terraform state and will henceforth be managed by Terraform.

4) Run `terraform plan`:

* If you see "No changes" -- the import was perfect

        No changes. Your infrastructure matches the configuration.

* If you see changes -- your config does not match reality. Update your config to match, then plan again until you get "No changes"

5) Run `terraform state list` -- the imported bucket should now appear alongside your other resources

- terraform state list

        data.aws_ami.amazon_linux
        data.aws_availability_zones.available
        aws_instance.main
        aws_internet_gateway.gw
        aws_route_table.example
        aws_route_table_association.example
        aws_s3_bucket.app_logs
        aws_s3_bucket.imported
        aws_security_group.main
        aws_subnet.main
        aws_vpc.main
        
Document: What is the difference between `terraform import` and creating a resource from scratch?

| terraform import                                                                                                  | Creating from scratch     |
|-------------------------------------------------------------------------------------------------------------------|---------------------------|
| Resource already exists in AWS                                                                                    | Resource doesnt exist yet |
| Terraform adopts the existing resource	|Terraform creates the resource                                                                 |
| Does not create the resource	|terraform apply creates it                                                           |                         
| Adds the resource to Terraform state	|Terraform creates it and records it in state                                 |                         
| Configuration must be written to match the existing resource	|Configuration describes what Terraform should create |                           
