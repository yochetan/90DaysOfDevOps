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



Write this in your own words -- not copy-pasted definitions.
