Task 1: Multi-Job Workflow

Create .github/workflows/multi-job.yml with 3 jobs:

* build — prints "Building the app"
* test — prints "Running tests"
* deploy — prints "Deploying"

`multi-job.yml`

    name: Multi-Jobs
    
    on: 
      workflow_dispatch:
    
    jobs:
      build:
        runs-on: ubuntu-latest
    
        steps:
          - name: building
            run: echo "Building the app"
    
      test:
        needs: build
        runs-on: ubuntu-latest
    
        steps:
          - name: testing
            run: echo "Running tests"
     
      deploy:
         needs: test
         runs-on: ubuntu-latest
    
         steps:
           - name: deploying
             run: echo "Deploying" 

Make test run only after build succeeds. Make deploy run only after test succeeds.

    yeah did.

Verify: Check the workflow graph in the Actions tab — does it show the dependency chain?
    
    yess it does shows


Task 2: Environment Variables

In a new workflow, use environment variables at 3 levels:

1) Workflow level — APP_NAME: myapp

2) Job level — ENVIRONMENT: staging

3) Step level — VERSION: 1.0.0

`env-context-demo.yml`

    name: Environment Variables and context demo
    
    on: 
      workflow_dispatch:
    
    env:
      APP_NAME: myapp
    
    jobs:
      demo:
        runs-on: ubuntu-latest
    
        env:
          ENVIRONMENT: staging
    
        steps:
          - name: Checkout repository
            uses: actions/checkout@v4
    
          - name: Print environment variables and Github context
            env:
              VERSION: 1.0.0
            run: |
              echo "Application: $APP_NAME"
              echo "Environment: $ENVIRONMENT"
              echo "Version: $VERSION"
    
              echo ""
              echo "GitHub Context"
              echo "Commit SHA: ${{ github.sha }}"
              echo "Triggered by: ${{ github.actor }}"

Print all three in a single step and verify each is accessible.

    echo "Application: $APP_NAME"
                  echo "Environment: $ENVIRONMENT"
                  echo "Version: $VERSION"

Then use a GitHub context variable — print the commit SHA and the actor (who triggered the run).

    echo "GitHub Context"
                  echo "Commit SHA: ${{ github.sha }}"
                  echo "Triggered by: ${{ github.actor }}"


Task 3: Job Outputs

1) Create a job that sets an output — e.g., today's date as a string

2) Create a second job that reads that output and prints it

3) Pass the value using outputs: and needs.<job>.outputs.<name>

`job-outputs.yml`

    name: Job Outputs
    
    on: 
      workflow_dispatch:
    
    jobs:
      job1:
        runs-on: ubuntu-latest
        outputs:
          today: ${{ steps.date.outputs.today }}
        steps:
          - name: Todays date in string
            id: date
            run: | 
              echo "today=$(date +'%Y-%m-%d')" >> $GITHUB_OUTPUT
    
      job2:
        needs: job1
        runs-on: ubuntu-latest
    
        steps:
          - name: Read and print the output
            run: |
              echo "${{ needs.job1.outputs.today }}"

* Syntax Summary 

| --- | --- |
|---|---|
| Purpose | Syntax |
| Set a step output | echo name=value >> $GITHUB_OUTPUT |
| Give the step an ID | id: date |
| Create a job output | outputs: today: ${{ steps.date.outputs.today }} |
| Make another job wait | needs: job1 |
| Read the job output | ${{ needs.generate-date.outputs.today }} |

- This demonstrates how to pass data between jobs using outputs and needs.<job>.outputs.<name>.

Write in your notes: Why would you pass outputs between jobs?

Imagine you have two jobs:

- Build the application.
- Deploy the application.

The build job might generate information that the deploy job needs, such as:

- A version number
- An image tag
- A commit hash
- A release ID
- The path or name of an artifact

Instead of recalculating or hardcoding these values, you pass them as job outputs.
