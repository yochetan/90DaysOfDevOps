Task 1: GitHub Secrets

1) Go to your repo → Settings → Secrets and Variables → Actions

        yeah did.

2) Create a secret called MY_SECRET_MESSAGE

        created.

3) Create a workflow that reads it and prints: The secret is set: true (never print the actual value)

        run: |
                          if [ -n "$MY_SECRET" ]; then
                            echo "The secret is set: true"
                          else
                            echo "The secret is set: false"
                          fi

4) Try to print ${{ secrets.MY_SECRET_MESSAGE }} directly — what does GitHub show?

        - name: Try to print the secret (for learning only)
                        run: |
                          echo "Secret value: ${{ secrets.MY_SECRET_MESSAGE }}"

It printed this:

        Secret value: ***

`secrets-demo.yml`

        name: Secrets Demo
        
        on:
          workflow_dispatch: 
        
        jobs:
          test-secret:
            runs-on: ubuntu-latest
        
            steps:
              - name: Check if secret exists
                env:
                  MY_SECRET: ${{ secrets.MY_SECRET_MESSAGE }}
                run: |
                  if [ -n "$MY_SECRET" ]; then
                    echo "The secret is set: true"
                  else
                    echo "The secret is set: false"
                  fi
        
              - name: Try to print the secret (for learning only)
                run: |
                  echo "Secret value: ${{ secrets.MY_SECRET_MESSAGE }}"

Write in your notes: Why should you never print secrets in CI logs?

a) CI logs may be visible to collaborators or anyone with repository access.

b) Secrets include API keys, passwords, tokens, cloud credentials, SSH keys, etc.

c) If exposed, an attacker could:

- Access your cloud infrastructure.
- Push malicious code.
- Deploy or delete applications.
- Read or modify databases.
- Impersonate your services.

d) Even though GitHub masks repository secrets, you should never intentionally print them because:

- Masking is not guaranteed for every transformed or partial value.
- Secrets copied into other variables or external outputs may not always be protected.
- Good security practice is to use secrets only where needed, not display them.


Task 2: Use Secrets as Environment Variables

1) Pass a secret to a step as an environment variable

2) Use it in a shell command without ever hardcoding it

3) Add DOCKER_USERNAME and DOCKER_TOKEN as secrets (you'll need these on Day 45)

`Docker-secrets.yml`

        name: Docker Secrets
        
        on:
          workflow_dispatch:
        
        jobs:
          use-secrets:
            runs-on: ubuntu-latest
        
            steps:
              - name: Pass secrets as environment variables
                env:
                  DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
                  DOCKER_TOKEN: ${{ secrets.DOCKER_TOKEN }}
                run: |
                  echo "Docker username is available."
        
                  if [ -n "$DOCKER_USERNAME" ]; then
                    echo "DOCKER_USERNAME is set."
                  fi
        
                  if [ -n "$DOCKER_TOKEN" ]; then
                    echo "DOCKER_TOKEN is set."
                  fi


Task 3: Upload Artifacts

1) Create a step that generates a file — e.g., a test report or a log file

        - name: Generate test report
                        run: |
                          mkdir reports
                          echo "=== Test Report ===" > reports/test-report.txt
                          echo "Workflow: ${{ github.workflow }}" >> reports/test-report.txt
                          echo "Branch: ${{ github.ref_name }}" >> reports/test-report.txt
                          echo "Commit: ${{ github.sha }}" >> reports/test-report.txt
                          echo "Generated at: $(date)" >> reports/test-report.txt

2) Use actions/upload-artifact to save it

        - name: Upload artifact
                        uses: actions/upload-artifact@v4
                        with:
                          name: test-report
                          path: reports/test-report.txt

3) After the workflow runs, download the artifact from the Actions tab

        Artifact download URL: https://github.com/yochetan/github-actions-practice/actions/runs/29960706866/artifacts/8545830959

`artifacts.yml`

        name: Artifacts
        
        on:
          workflow_dispatch: 
        
        jobs:
          generate-artifact:
            runs-on: ubuntu-latest
        
            steps:
              - uses: actions/checkout@v4
        
              - name: Generate test report
                run: |
                  mkdir reports
                  echo "=== Test Report ===" > reports/test-report.txt
                  echo "Workflow: ${{ github.workflow }}" >> reports/test-report.txt
                  echo "Branch: ${{ github.ref_name }}" >> reports/test-report.txt
                  echo "Commit: ${{ github.sha }}" >> reports/test-report.txt
                  echo "Generated at: $(date)" >> reports/test-report.txt
        
              - name: Display report
                run: cat reports/test-report.txt
        
              - name: Upload artifact
                uses: actions/upload-artifact@v4
                with:
                  name: test-report
                  path: reports/test-report.txt

Verify: Can you see and download it from GitHub?

        yeah its visible and downloadable 


Task 4: Download Artifacts Between Jobs

1) Job 1: generate a file and upload it as an artifact

        Job1:
                    runs-on: ubuntu-latest
                
                    steps:
                      - uses: actions/checkout@v4
                
                      - name: Generate a test report
                        run: |
                          mkdir reports
                          echo "This is a test report" > reports/test-report.txt
                 
                      - name: Upload artifact
                        uses: actions/upload-artifact@v4
                        with: 
                          name: test-report
                          path: reports/test-report.txt

2) Job 2: download the artifact from Job 1 and use it (print its contents)

        Job2:
                   runs-on: ubuntu-latest
                
                   needs: Job1
                
                   steps:
                     - name: Download artifact
                       uses: actions/download-artifact@v4
                       with:
                         name: test-report
                         path: downloaded-report
                
                     - name: Display artifact contents
                       run: |
                          echo "Downloaded files:"
                          ls -R downloaded-report
                          echo ""
                          echo "Report contents:"
                          cat downloaded-report/test-report.txt

`artifacts-between-jobs.yml`

        name: Download Artifacts Between Jobs
        
        on: 
          workflow_dispatch:
        
        jobs:
         Job1:
            runs-on: ubuntu-latest
        
            steps:
              - uses: actions/checkout@v4
        
              - name: Generate a test report
                run: |
                  mkdir reports
                  echo "This is a test report" > reports/test-report.txt
         
              - name: Upload artifact
                uses: actions/upload-artifact@v4
                with: 
                  name: test-report
                  path: reports/test-report.txt
        
         Job2:
           runs-on: ubuntu-latest
        
           needs: Job1
        
           steps:
             - name: Download artifact
               uses: actions/download-artifact@v4
               with:
                 name: test-report
                 path: downloaded-report
        
             - name: Display artifact contents
               run: |
                  echo "Downloaded files:"
                  ls -R downloaded-report
                  echo ""
                  echo "Report contents:"
                  cat downloaded-report/test-report.txt

Write in your notes: When would you use artifacts in a real pipeline?

Artifacts are useful whenever one job produces files that another job or a developer needs later. Common examples include:

- Sharing build outputs between jobs (build → deploy).
- Passing test reports or code coverage reports to later jobs.
- Storing application binaries (.jar, .exe, .zip, Docker image metadata).
- Saving logs or screenshots for debugging failed workflows.
- Preserving deployment packages for release or manual download.
- Keeping generated documentation or static website files.

Key point: Jobs run on separate runners with separate filesystems, so artifacts are the recommended way to persist and transfer files between jobs or after the workflow completes.


Task 5: Run Real Tests in CI

Take any script from your earlier days (Python or Shell) and run it in CI:

1) Add your script to the github-actions-practice repo

`maintenance.yml`

        #!/bin/bash
        
        echo "Starting maintenance..."
        
        echo "Checking disk usage..."
        df -h
        
        echo "Maintenance completed successfully."
        
        exit 0

2) Write a workflow that:

* Checks out the code
* Installs any dependencies needed
* Runs the script
* Fails the pipeline if the script exits with a non-zero code

`run-script.yml`

        name: Run Shell Script
        
        on:
          workflow_dispatch:
        
        jobs:
          run-script:
            runs-on: ubuntu-latest
        
            steps:
              - name: Checkout Repository
                uses: actions/checkout@v4
        
              - name: Make script executable
                run: chmod +x scripts/maintenance.sh
        
              - name: Run script
                run: ./scripts/maintenance.sh

3) Intentionally break the script — verify the pipeline goes red



4) Fix it — verify it goes green again


Task 6: Caching

1) Add actions/cache to a workflow that installs dependencies

2) Run it twice — observe the time difference

3) Write in your notes: What is being cached and where is it stored?
