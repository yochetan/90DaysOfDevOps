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

| Feature |Root Module |Child Module                                                                          |
|-----------------------------------------------------------------------------------------------------------  |
| Definition |The main Terraform configuration you execute |A reusable configuration called by another module |
| Location |Usually your main project directory |Usually a subdirectory or separate module source             |
| Execution |You run Terraform commands here |Used through a module block                                     |
| Purpose |Manages the overall infrastructure |Creates a specific part of the infrastructure                  |
| Inputs |Receives values from variables and .tfvars files |Receives values through module arguments          |
| Outputs |Displays useful infrastructure information |Exposes values to the root or calling module           |
