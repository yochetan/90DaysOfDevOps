Task 1: Learn Terraform Workspaces

1) Before building the project, understand workspaces:

        mkdir terraweek-capstone && cd terraweek-capstone
        terraform init
        
        # See current workspace
        terraform workspace show                    # default

        default

        # Create new workspaces
        terraform workspace new dev

        Created and switched to workspace "dev"!

        terraform workspace new staging

        Created and switched to workspace "staging"!

        terraform workspace new prod

        Created and switched to workspace "prod"!

        # List all workspaces
        terraform workspace list

          default
          dev
        * prod
          staging

        # Switch between them
        terraform workspace select dev

        Switched to workspace "dev".

        terraform workspace select staging

        Switched to workspace "staging".

        terraform workspace select prod

        Switched to workspace "prod".

Answer:

1) What does `terraform.workspace` return inside a config?

        terraform.workspace is a built-in Terraform expression that returns the name of the currently selected workspace.

2) Where does each workspace store its state file?

| Workspace | State file location                           |
|-----------|-----------------------------------------------|
| default   | terraform.tfstate                             |
| dev       | terraform.tfstate.d/dev/terraform.tfstate     |
| staging   | terraform.tfstate.d/staging/terraform.tfstate |
| prod      | terraform.tfstate.d/prod/terraform.tfstate    |

3) How is this different from using separate directories per environment?

| Feature                   | Terraform workspaces                                        | Separate directories                                            |
|---------------------------|-------------------------------------------------------------|-----------------------------------------------------------------|
| Configuration             | Same configuration files                                    | Each environment can have its own files                         |
| State                     | Separate state per workspace                                | Separate state per directory/backend configuration              |
| Code duplication          | Usually less                                                | Can be more if code is duplicated                               |
| Environment customization | Uses variables and workspace conditions                     | Easy to customize each environment independently                |
| Risk of mistakes          | Possible to select the wrong workspace                      | Possible to run Terraform in the wrong directory                |
| Best use                  | Similar environments with the same infrastructure structure | Environments needing different configurations or infrastructure |

---

Task 2: Set Up the Project Structure

Create this layout:
        
        terraweek-capstone/
          main.tf                   # Root module -- calls child modules
          variables.tf              # Root variables
          outputs.tf                # Root outputs
          providers.tf              # AWS provider and backend
          locals.tf                 # Local values using workspace
          dev.tfvars                # Dev environment values
          staging.tfvars            # Staging environment values
          prod.tfvars               # Prod environment values
          .gitignore                # Ignore state, .terraform, tfvars with secrets
          modules/
            vpc/
              main.tf
              variables.tf
              outputs.tf
            security-group/
              main.tf
              variables.tf
              outputs.tf
            ec2-instance/
              main.tf
              variables.tf
              outputs.tf

Create the `.gitignore`:

        .terraform/
        *.tfstate
        *.tfstate.backup
        *.tfvars
        .terraform.lock.hcl

*Document*: Why is this file structure considered best practice?

        This file structure is considered best practice because it makes your Terraform project modular, reusable, organized, and easier to maintain across multiple environments such as dev, staging, and prod.

---

Task 3: Build the Custom Modules

Create three focused modules:

Module 1: `modules/vpc/`

* Input: `cidr`, `public_subnet_cidr`, `environment`, `project_name`

* Resources: VPC, public subnet, internet gateway, route table, route table association

* Output: `vpc_id`, `subnet_id`

* All resources tagged with environment and project name

Module 2: `modules/security-group/`

* Input: `vpc_id`, `ingress_ports`, `environment`, `project_name`

* Resources: Security group with dynamic ingress rules, allow all egress

* Output: `sg_id`

Module 3: `modules/ec2-instance/`

* Input: `ami_id`, `instance_type`, `subnet_id`, `security_group_ids`, `environment`, `project_name`

* Resources: EC2 instance with tags

* Output: `instance_id`, `public_ip`

Write and validate each module:

terraform validate
