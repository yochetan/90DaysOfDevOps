Task 1: GitHub-Hosted Runners

1) Create a workflow with 3 jobs, each on a different OS:

- ubuntu-latest
- windows-latest
- macos-latest

2) In each job, print:

- The OS name
- The runner's hostname
- The current user running the job

3) Watch all 3 run in parallel

`github-hosted-runners.yml`

  name: Github Hosted Runners
  
  on:
    workflow_dispatch: 
  
  jobs:
  
    Ubuntu:
      runs-on: ubuntu-latest
  
      steps:
        - name: Print Runner OS
          run: |
            echo "Runner OS: $RUNNER_OS"
        
        - name: Print Hostname
          run: hostname
  
        - name: Print Current User
          run: whoami
  
    MAC:
      runs-on: macos-latest
  
      steps:
        - name: Print Runner OS
          run: |
            echo "Runner OS: $RUNNER_OS"
  
        - name: Print Hostname
          run: hostname
  
        - name: Print Current User
          run: whoami
  
    Windows:
      runs-on: windows-latest
  
      steps:
        - name: Print Runner OS
          run: |
            echo "Runner OS: $RUNNER_OS"


Write in your notes: What is a GitHub-hosted runner? Who manages it?

    Github-hosted-runner is github own runner which provide some limited time services for a month
    and it is managed by the github itself.

    A GitHub-hosted runner is a temporary virtual machine (VM) provided by GitHub to execute your workflow jobs. Every workflow job gets a fresh, clean environment with common development tools already installed.

    GitHub is responsible for:

    Creating the virtual machine.
    Installing the operating system.
    Maintaining and updating the software.
    Cleaning up and deleting the VM after the job finishes.


Task 2: Explore What's Pre-installed

1) On the ubuntu-latest runner, run a step that prints:

* Docker version
* Python version
* Node version
* Git version

`version.yml`

    name: Check version of softwares
    
    on:
      push:
        branches:
          - main
      workflow_dispatch:
    
    jobs:
      ubuntu-tools:
        runs-on: ubuntu-latest
    
        steps:
          - name: Check Docker version
            run: docker --version
         
          - name: Check node version
            run: node --version
        
          - name: Check git version
            run: git --version

2) Look up the GitHub docs for the full list of pre-installed software on ubuntu-latest

        Docker and Docker Compose
        Python (multiple versions)
        Node.js
        Git
        Java (JDK)
        .NET SDK
        Go
        Ruby
        PHP
        Rust
        C/C++ compilers
        Azure CLI
        AWS CLI
        Google Cloud CLI
        Terraform
        GitHub CLI (gh)
        Maven
        Gradle
        npm, yarn, pnpm
        kubectl, Helm
        PowerShell
        Many other development and DevOps tools

Write in your notes: Why does it matter that runners come with tools pre-installed?

- Faster execution: No need to install common tools every run.
- Simpler workflows: Less setup code in your YAML files.
- Consistent builds: Every job starts from a clean, standardized environment.
- Lower maintenance: GitHub manages and updates the runner images.
- Ready for many projects: Common languages and DevOps tools are available out of the box.


Task 3: Set Up a Self-Hosted Runner

1) Go to your GitHub repo → Settings → Actions → Runners → New self-hosted runner

        done

2) Choose Linux as the OS

        done

3) Follow the instructions to download and configure the runner on:
* Your local machine, OR

* A cloud VM (EC2, Utho, or any VPS)

4) Start the runner — verify it shows as Idle in GitHub

Verify: Your runner appears in the Runners list with a green dot.
