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

---

Task 4: Create Your Inventory File

The inventory tells Ansible which servers to manage. Create a project directory and your first inventory:

        mkdir ansible-practice && cd ansible-practice

Create a file called `inventory.ini`:

        [web]
        web-server ansible_host=54.186.239.88 ansible_user=ec2-user
        
        [app]
        app-server ansible_host=32.191.220.252 ansible_user=ubuntu
        
        [db]
        db-server ansible_host=34.220.233.127 ansible_user=ec2-user
        
        [all:vars]
        ansible_ssh_private_key_file=/home/ubuntu/ansible-practice/keys/terra-automate-key
        ansible_host_key_checking=false

Verify Ansible can reach all hosts:

* ansible all -i inventory.ini -m ping

        [WARNING]: Host 'web-server' is using the discovered Python interpreter at '/usr/bin/python3.9', but future installation of another Python interpreter could cause a different interpreter to be discovered. See https://docs.ansible.com/ansible-core/2.20/reference_appendices/interpreter_discovery.html for more information.
        web-server | SUCCESS => {
            "ansible_facts": {
                "discovered_interpreter_python": "/usr/bin/python3.9"
            },
            "changed": false,
            "ping": "pong"
        }
        [WARNING]: Host 'db-server' is using the discovered Python interpreter at '/usr/bin/python3.12', but future installation of another Python interpreter could cause a different interpreter to be discovered. See https://docs.ansible.com/ansible-core/2.20/reference_appendices/interpreter_discovery.html for more information.
        db-server | SUCCESS => {
            "ansible_facts": {
                "discovered_interpreter_python": "/usr/bin/python3.12"
            },
            "changed": false,
            "ping": "pong"
        }
        [WARNING]: Host 'app-server' is using the discovered Python interpreter at '/usr/bin/python3.14', but future installation of another Python interpreter could cause a different interpreter to be discovered. See https://docs.ansible.com/ansible-core/2.20/reference_appendices/interpreter_discovery.html for more information.
        app-server | SUCCESS => {
            "ansible_facts": {
                "discovered_interpreter_python": "/usr/bin/python3.14"
            },
            "changed": false,
            "ping": "pong"
        }

You should see green `SUCCESS` with `"ping": "pong"` for each host.

Troubleshoot: If ping fails:

* Check the SSH key path and permissions (`chmod 400 your-key.pem`)

* Check the security group allows SSH from your IP

* Check the `ansible_user` matches your AMI (ec2-user for Amazon Linux, ubuntu for Ubuntu)

---

Task 5: Run Ad-Hoc Commands

Ad-hoc commands let you run quick one-off tasks without writing a playbook.

1) Check uptime on all servers:

* ansible all -i inventory.ini -m command -a "uptime"

        app-server | CHANGED | rc=0 >>
         21:25:54 up  1:04,  1 user,  load average: 0.08, 0.02, 0.00
        web-server | CHANGED | rc=0 >>
         21:25:54 up  1:04,  1 user,  load average: 0.00, 0.00, 0.00
        db-server | CHANGED | rc=0 >>
         21:25:54 up  1:03,  1 user,  load average: 0.07, 0.03, 0.00

2) Check free memory on web servers only:

* ansible web -i inventory.ini -m command -a "free -h"

        web-server | CHANGED | rc=0 >>
                       total        used        free      shared  buff/cache   available
        Mem:           913Mi       166Mi       496Mi       0.0Ki       250Mi       615Mi
        Swap:             0B          0B          0B

3) Check disk space on all servers:

* ansible all -i inventory.ini -m command -a "df -h"

        app-server | CHANGED | rc=0 >>
        Filesystem       Size  Used Avail Use% Mounted on
        /dev/root        8.6G  2.1G  6.6G  24% /
        tmpfs            455M     0  455M   0% /dev/shm
        tmpfs            182M  888K  181M   1% /run
        efivarfs         128K  3.1K  120K   3% /sys/firmware/efi/efivars
        tmpfs            455M  120K  455M   1% /tmp
        none             1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
        none             1.0M     0  1.0M   0% /run/credentials/systemd-resolved.service
        /dev/nvme0n1p13  989M   96M  827M  11% /boot
        /dev/nvme0n1p15  105M  6.3M   99M   7% /boot/efi
        none             1.0M     0  1.0M   0% /run/credentials/systemd-networkd.service
        none             1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
        none             1.0M     0  1.0M   0% /run/credentials/serial-getty@ttyS0.service
        tmpfs             91M  8.0K   91M   1% /run/user/1000
        web-server | CHANGED | rc=0 >>
        Filesystem        Size  Used Avail Use% Mounted on
        devtmpfs          4.0M     0  4.0M   0% /dev
        tmpfs             457M     0  457M   0% /dev/shm
        tmpfs             183M  436K  183M   1% /run
        efivarfs          128K  2.7K  121K   3% /sys/firmware/efi/efivars
        /dev/nvme0n1p1     10G  1.6G  8.4G  16% /
        tmpfs             457M  120K  457M   1% /tmp
        /dev/nvme0n1p128   10M  1.3M  8.7M  13% /boot/efi
        tmpfs              92M     0   92M   0% /run/user/1000
        db-server | CHANGED | rc=0 >>
        Filesystem      Size  Used Avail Use% Mounted on
        /dev/nvme0n1p3  9.8G  2.1G  7.8G  21% /
        devtmpfs        347M     0  347M   0% /dev
        tmpfs           454M     0  454M   0% /dev/shm
        efivarfs        128K  3.1K  120K   3% /sys/firmware/efi/efivars
        tmpfs           182M  3.5M  178M   2% /run
        tmpfs           1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
        /dev/nvme0n1p2  200M  9.0M  191M   5% /boot/efi
        tmpfs           1.0M     0  1.0M   0% /run/credentials/serial-getty@ttyS0.service
        tmpfs           1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
        tmpfs            91M  4.0K   91M   1% /run/user/1000

4) Install a package on the web group:

* ansible web -i inventory.ini -m yum -a "name=git state=present" --become

        web-server | CHANGED => {
            "ansible_facts": {
                "pkg_mgr": "dnf"
            },
            "changed": true,
            "msg": "",
            "rc": 0,
            "results": [
                "Installed: perl-lib-0.65-477.amzn2023.0.10.x86_64",
                "Installed: git-core-2.50.1-1.amzn2023.0.1.x86_64",
                "Installed: git-2.50.1-1.amzn2023.0.1.x86_64",
                "Installed: perl-Error-1:0.17030-2.amzn2023.0.1.noarch",
                "Installed: git-core-doc-2.50.1-1.amzn2023.0.1.noarch",
                "Installed: perl-TermReadKey-2.38-9.amzn2023.0.3.x86_64",
                "Installed: perl-Git-2.50.1-1.amzn2023.0.1.noarch",
                "Installed: perl-File-Find-1.37-477.amzn2023.0.10.noarch"
            ]
        }

(Use `apt` instead of `yum` if running Ubuntu)

5) Copy a file to all servers:

* echo "Hello from Ansible" > hello.txt

* ansible all -i inventory.ini -m copy -a "src=hello.txt dest=/tmp/hello.txt"

        web-server | CHANGED => {
            "changed": true,
            "checksum": "9b02adc484350380842c4c735d57a1ad071ff0a1",
            "dest": "/tmp/hello.txt",
            "gid": 1000,
            "group": "ec2-user",
            "md5sum": "914d64e550abfbc035d79d50e214e47d",
            "mode": "0664",
            "owner": "ec2-user",
            "secontext": "unconfined_u:object_r:user_tmp_t:s0",
            "size": 19,
            "src": "/home/ec2-user/.ansible/tmp/ansible-tmp-1793827748.1027434-2740-215017212611156/.source.txt",
            "state": "file",
            "uid": 1000
        }
        db-server | CHANGED => {
            "changed": true,
            "checksum": "9b02adc484350380842c4c735d57a1ad071ff0a1",
            "dest": "/tmp/hello.txt",
            "gid": 1000,
            "group": "ec2-user",
            "md5sum": "914d64e550abfbc035d79d50e214e47d",
            "mode": "0644",
            "owner": "ec2-user",
            "secontext": "unconfined_u:object_r:user_home_t:s0",
            "size": 19,
            "src": "/home/ec2-user/.ansible/tmp/ansible-tmp-1793827748.2542653-2742-243417181174135/.source.txt",
            "state": "file",
            "uid": 1000
        }
        app-server | CHANGED => {
            "changed": true,
            "checksum": "9b02adc484350380842c4c735d57a1ad071ff0a1",
            "dest": "/tmp/hello.txt",
            "gid": 1000,
            "group": "ubuntu",
            "md5sum": "914d64e550abfbc035d79d50e214e47d",
            "mode": "0664",
            "owner": "ubuntu",
            "size": 19,
            "src": "/home/ubuntu/.ansible/tmp/ansible-tmp-1793827748.579392-2741-10215782980065/.source.txt",
            "state": "file",
            "uid": 1000
        }

6) Verify the file was copied:

* ansible all -i inventory.ini -m command -a "cat /tmp/hello.txt"


        app-server | CHANGED | rc=0 >>
        Hello from Ansible
        web-server | CHANGED | rc=0 >>
        Hello from Ansible
        db-server | CHANGED | rc=0 >>
        Hello from Ansible

Document: What does `--become` do? When do you need it?

        --become tells Ansible to execute tasks with elevated privileges, usually as the root user.

---

Task 6: Explore Inventory Groups and Patterns

1) Create a group of groups -- add this to your `inventory.ini`:

        [application:children]
        web
        app
        
        [all_servers:children]
        application
        db

2) Run commands against different groups:

* ansible application -i inventory.ini -m ping     # web + app servers

        web-server | SUCCESS => {
            "changed": false,
            "ping": "pong"
        }
        app-server | SUCCESS => {
            "changed": false,
            "ping": "pong"
        }

* ansible db -i inventory.ini -m ping               # only db server

        db-server | SUCCESS => {
            "changed": false,
            "ping": "pong"
        }

* ansible all_servers -i inventory.ini -m ping      # everything

        app-server | SUCCESS => {
            "changed": false,
            "ping": "pong"
        }
        web-server | SUCCESS => {
            "changed": false,
            "ping": "pong"
        }
        db-server | SUCCESS => {
            "changed": false,
            "ping": "pong"
        }

3) Use patterns:

* ansible 'web:app' -i inventory.ini -m ping        # OR: web or app

        app-server | SUCCESS => {
            "changed": false,
            "ping": "pong"
        }
        web-server | SUCCESS => {
            "changed": false,
            "ping": "pong"
        }

* ansible 'all:!db' -i inventory.ini -m ping        # NOT: all except db

        app-server | SUCCESS => {
            "changed": false,
            "ping": "pong"
        }
        web-server | SUCCESS => {
            "changed": false,
            "ping": "pong"
        }

4) Create an `ansible.cfg` to avoid typing `-i inventory.ini` every time:

        [defaults]
        inventory = inventory.ini
        host_key_checking = False
        remote_user = ubuntu
        private_key_file = /home/ubuntu/ansible-practice/keys/terra-automate-key

Now you can simply run:

* ansible all -m ping

        web-server | SUCCESS => {
            "changed": false,
            "ping": "pong"
        }
        db-server | SUCCESS => {
            "changed": false,
            "ping": "pong"
        }
        app-server | SUCCESS => {
            "changed": false,
            "ping": "pong"
        }

Verify: Does `ansible all -m ping` work without specifying the inventory file?

        Yes, ansible all -m ping can work without specifying -i inventory.ini, but only if Ansible can find your inventory automatically.

