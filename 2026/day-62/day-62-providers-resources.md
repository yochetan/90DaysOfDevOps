Task 1: Explore the AWS Provider

1) Create a new project directory: `terraform-aws-infra`

        Created!!

2) Write a `providers.tf` file:

- Define the `terraform` block with `required_providers` pinning the AWS provider to version `~> 5.0`

- Define the `provider "aws"` block with your region

3) Run `terraform init` and check the output -- what version was installed?

4) Read the provider lock file `.terraform.lock.hcl` -- what does it do?

Document: What does `~> 5.0` mean? How is it different from `>= 5.0` and `= 5.0.0`?

