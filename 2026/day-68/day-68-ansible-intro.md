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
