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

