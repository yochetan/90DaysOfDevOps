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

3) What does "agentless" mean? How does Ansible connect to managed nodes?

4) Draw or describe the Ansible architecture:

* Control Node -- the machine where Ansible runs (your laptop or a jump server)

* Managed Nodes -- the servers Ansible configures (your EC2 instances)

* Inventory -- the list of managed nodes

* Modules -- units of work Ansible executes (install a package, copy a file, start a service)

* Playbooks -- YAML files that define what to do on which hosts
