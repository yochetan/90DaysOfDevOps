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

Print all three in a single step and verify each is accessible.

Then use a GitHub context variable — print the commit SHA and the actor (who triggered the run).

