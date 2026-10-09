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
