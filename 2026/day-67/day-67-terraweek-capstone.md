Task 1: Learn Terraform Workspaces

1) Before building the project, understand workspaces:

        mkdir terraweek-capstone && cd terraweek-capstone
        terraform init
        
        # See current workspace
        terraform workspace show                    # default

        default

        # Create new workspaces
        terraform workspace new dev
        terraform workspace new staging
        terraform workspace new prod
        
        # List all workspaces
        terraform workspace list
        
        # Switch between them
        terraform workspace select dev
        terraform workspace select staging
        terraform workspace select prod

Answer:

1) What does `terraform.workspace` return inside a config?

2) Where does each workspace store its state file?

3) How is this different from using separate directories per environment?

---
