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

2) `main.tf` -- define the resource:

* `aws_security_group` in the given VPC
* Use `dynamic "ingress"` block to create rules from the `ingress_ports` list
* Allow all egress

`modules/security-group/main.tf`

3) `outputs.tf` -- expose:

* `sg_id`

`modules/security-group/outputs.tf`

This is your first time using a `dynamic` block -- it loops over a list to generate repeated nested blocks.
