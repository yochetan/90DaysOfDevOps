Task 1: Inspect Your Current State

Use your Day 63 config (or create a small config with a VPC and EC2 instance). Apply it and then explore the state:

        terraform show                                    # Full state in human-readable format
        terraform state list                              # All resources tracked by Terraform
        terraform state show aws_instance.<name>          # Every attribute of the instance
        terraform state show aws_vpc.<name>               # Every attribute of the VPC

Answer:

1) How many resources does Terraform track?

2) What attributes does the state store for an EC2 instance? (hint: way more than what you defined)

3) Open `terraform.tfstate` in an editor -- find the `serial` number. What does it represent?
