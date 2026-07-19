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

      selected an EC2 runner

4) Start the runner — verify it shows as Idle in GitHub

        yeah, i did and it showed idle.

Verify: Your runner appears in the Runners list with a green dot.

    yeahhhh!


Task 4: Use Your Self-Hosted Runner

1) Create .github/workflows/self-hosted.yml

        created.

2) Set runs-on: self-hosted

        yeah did.

3) Add steps that:

* Print the hostname of the machine (it should be YOUR machine/VM)
* Print the working directory
* Create a file and verify it exists on your machine after the run

`self-hosted.yml`

    name: Self Hosted Runner
    
    on:
      workflow_dispatch: 
    
    jobs:
      Task-on-self-hosted-runner:
        runs-on: self-hosted
    
        steps:
          - name: Print Hostname
            run: hostname
    
          - name: Print current directory
            run: pwd
            
          - name: Create a file
            run: |
              echo "Hello from self-hosted runner!" > self-hosted-test.txt

4) Trigger it and watch it run on your own hardware

        yeah it ran it and got the outputs.

Verify: Check your machine — is the file there?

    yess.

Task 5: Labels

1) Add a label to your self-hosted runner (e.g., my-linux-runner)

        yeah did, gave it: my-linux-runner

2) Update your workflow to use runs-on: [self-hosted, my-linux-runner]

        runs-on:
          self-hosted
          Linux
          my-linux-runner

3) Trigger it — does it still pick up the job?

        YES IT DOES

Write in your notes: Why are labels useful when you have multiple self-hosted runners?

- They allow GitHub to select the appropriate runner automatically.
- They ensure jobs run on machines with the required environment.
- They help organize and manage many runners efficiently.
- They make workflows more reliable and easier to maintain.


Task 6: GitHub-Hosted vs Self-Hosted

Fill this in your notes:

| Feature | GitHub-Hosted Runner | Self-Hosted Runner |
|---|---|---|
| Who manages it? | GitHub | You (or your organization) |
| Cost | Included with GitHub minutes (extra usage may incur charges depending on your plan) | You pay for and maintain the machine or VM |
| Pre-installed tools | Many common tools (Docker, Git, Python, Node.js, Java, etc.) are already installed | You install and maintain the tools you need |
| Good for | General CI/CD, open-source projects, quick setup, and standard builds | Custom software, private networks, special hardware (GPU), large builds, or company infrastructure |
| Security concern | Code runs on temporary GitHub-managed VMs that are destroyed after each job | Your machine is exposed to workflow jobs, so you must secure, update, and monitor it |
