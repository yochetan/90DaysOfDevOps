Task 1: Understand Ansible

Research and write short notes on:

1) What is configuration management? Why do we need it?

        Configuration Management (CM) is the process of automatically managing, configuring, maintaining, and updating servers and systems to ensure they remain in a desired state.

        In simple words, configuration management helps us manage multiple servers without manually configuring each one.
        
        Example:
        
        Suppose you have 10 AWS EC2 instances. You need to:
        - Install Nginx on all 10 servers.
        - Configure firewall rules.
        - Create users and directories.
        - Start and enable services.
        - Keep software versions consistent.
        Instead of connecting to each EC2 instance manually, you can use Ansible to automate all these tasks.
        
        Why do we need Configuration Management?
        1. Automation: Reduces repetitive manual work.
        2. Consistency: Maintains the same configuration across servers.
        3. Time-saving: Configures multiple servers simultaneously.
        4. Error reduction: Minimizes human mistakes.
        5. Scalability: Easily manages hundreds or thousands of servers.
        6. Reliability: Keeps infrastructure in the desired state.
        Real-world DevOps example: When a new EC2 instance is launched, Ansible can automatically install Docker, configure users, and start required services.

2) How is Ansible different from Chef, Puppet, and Salt?

        All four are Configuration Management tools used to automate server configuration, but they differ in architecture and approach.

| Feature        | Ansible     | Chef         | Puppet       | SaltStack           |
|----------------|-------------|--------------|--------------|---------------------|
| Architecture   | Agentless   | Agent-based  | Agent-based  | Usually agent-based |
| Language       | YAML        | Ruby         | Puppet DSL   | YAML / Python       |
| Communication  | SSH / WinRM | Chef Client  | Puppet Agent | ZeroMQ / SSH        |
| Setup          | Easy        | More complex | More complex | Moderate            |
| Learning curve | Easy        | High         | Moderate     | Moderate            |
| Execution      | Push-based  | Pull-based   | Pull-based   | Push and pull       |


3) What does "agentless" mean? How does Ansible connect to managed nodes?

        Agentless means Ansible does not normally require a separate agent or background software to be installed on every managed server.
        
        For example:
        - In Ansible, you install Ansible on your control node.
        - You do not normally install Ansible on your EC2 instances.
        - Ansible connects remotely using SSH.
        How does Ansible connect?
        
        For Linux servers, Ansible uses:
        - SSH (Secure Shell): Establishes a secure connection.
        - Port 22: Default SSH port.
        - Private SSH key: Authenticates the control node.
        - Python: Usually required on managed Linux nodes to execute modules.
        For Windows servers, Ansible can use WinRM or SSH.
        
        Example with AWS EC2:
        
        Suppose you have:
        - Control Node: Ubuntu laptop.
        - Managed Node: Amazon Linux EC2 instance.
        - SSH Key: my-key.pem.

4) Draw or describe the Ansible architecture:

* Control Node -- the machine where Ansible runs (your laptop or a jump server)

* Managed Nodes -- the servers Ansible configures (your EC2 instances)

* Inventory -- the list of managed nodes

* Modules -- units of work Ansible executes (install a package, copy a file, start a service)

* Playbooks -- YAML files that define what to do on which hosts

| Component     | Meaning                                         | Example           |
|---------------|-------------------------------------------------|-------------------|
| Control Node  | Machine where Ansible is installed and executed | Ubuntu laptop     |
| Managed Nodes | Servers configured by Ansible                   | AWS EC2 instances |
| Inventory     | List of managed servers                         | inventory.ini     |
| Modules       | Perform individual tasks                        | Install Nginx     |
| Playbooks     | YAML files containing automation instructions   | install-nginx.yml |

<img width="1772" height="1204" alt="mermaid-diagram" src="https://github.com/user-attachments/assets/1c67f71b-cd17-4f16-ad1a-0780fc05f3ed" />

---

Task 2: Set Up Your Lab Environment

You need 2-3 EC2 instances to practice on. Choose one approach:

Option A: Use Terraform (recommended -- you just learned this) Use your TerraWeek skills to provision 3 EC2 instances with:

* Amazon Linux 2 or Ubuntu 22.04

* `t2.micro` instance type

* A security group allowing SSH (port 22)

* A key pair for SSH access

`ec2.tf`

        # Region
        provider "aws" {
          region = "us-west-2"
        }
        
        # Key Pair
        resource "aws_key_pair" "my_key_pair" {
          key_name   = "terra-automate-key"
          public_key = file("${path.module}/terra-automate-key.pub")
        }
        
        # Default VPC
        resource "aws_default_vpc" "default" {}
        
        # Security Group
        resource "aws_security_group" "my_security_group" {
          name        = "terra-security-group"
          vpc_id      = aws_default_vpc.default.id # interpolation
          description = "Inbound and outbound rules for the instance security group"
        }
        
        # Inbound & Outbound rules
        resource "aws_vpc_security_group_ingress_rule" "allow_http" {
          security_group_id = aws_security_group.my_security_group.id
          cidr_ipv4         = "0.0.0.0/0"
          from_port         = 80
          ip_protocol       = "tcp"
          to_port           = 80
        }
        
        resource "aws_vpc_security_group_ingress_rule" "allow_ssh" {
          security_group_id = aws_security_group.my_security_group.id
          cidr_ipv4         = "0.0.0.0/0"
          from_port         = 22
          ip_protocol       = "tcp"
          to_port           = 22
        }
        
        resource "aws_vpc_security_group_egress_rule" "allow_all_traffic" {
          security_group_id = aws_security_group.my_security_group.id
          cidr_ipv4         = "0.0.0.0/0"
          ip_protocol       = "-1" # all protocols and ports
        }
        
        # EC2 Instances
        resource "aws_instance" "my_instance" {
        
          for_each = var.instances
        
          ami           = each.value.ami
          instance_type = each.value.instance_type
        
          key_name               = aws_key_pair.my_key_pair.key_name
          vpc_security_group_ids = [aws_security_group.my_security_group.id]
        
          root_block_device {
            volume_size = 10
            volume_type = "gp3"
          }
        
          tags = {
            Name      = each.key
            OS_Family = each.value.os_family
          }
        }

`variables.tf`

        variable "instances" {
        
          description = "Map of instance names to AMI IDs, SSH users, and OS Family"
        
          type = map(object({
            ami           = string
            user          = string
            os_family     = string
            instance_type = string
          }))
        
          default = {
            "control-node-ubuntu" = {
              ami           = "ami-02167eae61967e403"
              user          = "ubuntu"
              os_family     = "ubuntu"
              instance_type = "t3.micro"
            }
            "worker-ubuntu" = {
              ami           = "ami-02167eae61967e403"
              user          = "ubuntu"
              os_family     = "ubuntu"
              instance_type = "t3.micro"
            }
            "worker-redhat" = {
              ami           = "ami-0da467f007dfebd6b"
              user          = "ec2-user"
              os_family     = "redhat"
              instance_type = "t3.micro"
            }
            "worker-amazon" = {
              ami           = "ami-0803a33fd43db1fbf"
              user          = "ec2-user"
              os_family     = "amazon"
              instance_type = "t3.micro"
            }
          }
        }

Verify you can SSH into each one from your control node:

        ssh -i ~/your-key.pem ec2-user@<public-ip-1>
        ssh -i ~/your-key.pem ec2-user@<public-ip-2>
        ssh -i ~/your-key.pem ec2-user@<public-ip-3>

---

Task 3: Install Ansible

Install Ansible on your control node (your laptop or one dedicated EC2 instance):

# Ubuntu/Debian

* sudo apt update

* sudo apt install ansible -y

# Verify

* ansible --version

        ansible [core 2.20.1]
          config file = None
          configured module search path = ['/home/ubuntu/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
          ansible python module location = /usr/lib/python3/dist-packages/ansible
          ansible collection location = /home/ubuntu/.ansible/collections:/usr/share/ansible/collections
          executable location = /usr/bin/ansible
          python version = 3.14.4 (main, Apr  8 2026, 04:02:31) [GCC 15.2.0] (/usr/bin/python3)
          jinja version = 3.1.6
          pyyaml version = 6.0.3 (with libyaml v0.2.5)

- Confirm the output shows the Ansible version, config file path, and Python version.

        yes, it does.

Document: On which machine did you install Ansible? Why is it only needed on the control node?

        Ansible is installed on the Control Node, which is a jump server.

        Ansible is only needed on the control node because it manages all the automation tasks and connects remotely to managed nodes using SSH.
