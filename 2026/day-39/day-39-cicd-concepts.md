Task 1: The Problem

Think about a team of 5 developers all pushing code to the same repo manually deploying to production.

Write in your notes:

1) What can go wrong?
        
        - No proper testing and that leaves bug and deployment risks and lot of time wasted.
        - Developers may overwrite each other's changes.
        - Bugs can be introduced into production.
        - Deployment conflicts can occur when multiple developers deploy at the same time.

2) What does "it works on my machine" mean and why is it a real problem?

        - The dependencies are on the developer pc and not on the client/user which leads to bad 
        - Different operating systems
        - Different software versions
        - Missing dependencies
        - Different environment variables or configurations

3) How many times a day can a team safely deploy manually?

        - very few cause doing all manually takes time
        - There is no fixed limit, but manual deployments are slow and error-prone. Most teams can safely perform only a few manual deployments per day because each deployment requires careful checking and human intervention.
        - With CI/CD automation, teams can safely deploy many times per day (sometimes dozens or even hundreds of times) because testing and deployment steps are automated and consistent.

* Conclusion: Manual deployments increase risk and reduce deployment frequency, while automated CI/CD pipelines make deployments faster, safer, and more reliable.


Task 2: CI vs CD

Research and write short definitions (2-3 lines each):

1) Continuous Integration — what happens, how often, what it catches

        automates - code integration changes into a shared repo, testing, merging code 
        Example: A developer pushes code to GitHub, and a GitHub Actions workflow automatically builds the project and runs unit tests.

2) Continuous Delivery — how it's different from CI, what "delivery" means

        automates - delivery of applications to selected environments(like staging) and ensuring code reliably which can be released anytime
        Example: After CI tests pass, a pipeline automatically deploys the application to a staging environment, and a manager clicks "Approve" before releasing it to production.

3) Continuous Deployment — how it differs from Delivery, when teams use it
        
        automates - deployment of every change that passes the tests directly to production without human intervention
        Example: A change is pushed to the main branch, passes all tests, and is automatically deployed to production through a pipeline.

Write one real-world example for each.

*QUICK OVERVIEW AND COMPARISON*

        | | Practice               | Main Goal                      | Deployment to Production | |
        |----------------------------------------------------------------------------------------|
        | | ---------------------- | ------------------------------ | ------------------------ | |
        | | Continuous Integration | Build and test code frequently | Not included             | |
        | | Continuous Delivery    | Keep code ready for release    | Manual approval required | |
        | | Continuous Deployment  | Fully automate releases        | Automatic deployment     | |


Task 3: Pipeline Anatomy

A pipeline has these parts — write what each one does:

- Trigger — what starts the pipeline

        A trigger is the event that starts a pipeline. Common triggers include pushing code to a repository, creating a pull request, scheduling a run, or manually starting the pipeline.
        
        Example: A developer pushes code to the main branch.

- Stage — a logical phase (build, test, deploy)

        A stage is a logical phase of the pipeline that groups related jobs together. Typical stages are Build, Test, and Deploy.
        
        Example: A "Test" stage runs all automated tests after the application is built.

- Job — a unit of work inside a stage

        A job is a collection of steps executed on the same runner. Multiple jobs can exist within a stage and may run in parallel.
        
        Example: A "Unit Test" job runs all unit tests for the application.

- Step — a single command or action inside a job

        A step is a single command or action within a job. Steps are executed sequentially.
        
        Example: Running npm install or docker build -t myapp .

- Runner — the machine that executes the job

        A runner is the machine (physical, virtual, or containerized) that executes pipeline jobs.
        
        Example: A GitHub-hosted Ubuntu runner executes a workflow job.

- Artifact — output produced by a job

        An artifact is a file or output produced by a job and stored for later use by other jobs or deployments.
        
        Example: A compiled application package, Docker image, test report, or build log.


Task 4: Draw a Pipeline

Draw a CI/CD pipeline for this scenario:

> A developer pushes code to GitHub. The app is tested, built into a Docker image, and deployed to a staging server.

Include at least 3 stages. Hand-drawn and photographed is perfectly fine.

         CI/CD PIPELINE
        
        ┌───────────┐
        │ Developer │
        └─────┬─────┘
              │ Push Code
              ▼
        ┌───────────┐
        │  GitHub   │
        └─────┬─────┘
              │ Trigger Pipeline
              ▼
        
        ┌──────────────────────────────┐
        │ STAGE 1: TEST                │
        ├──────────────────────────────┤
        │ • Checkout code              │
        │ • Install dependencies       │
        │ • Run unit tests             │
        │ • Generate test reports      │
        └──────────────┬───────────────┘
                       │ Success
                       ▼
        
        ┌──────────────────────────────┐
        │ STAGE 2: BUILD               │
        ├──────────────────────────────┤
        │ • Build application          │
        │ • Build Docker image         │
        │ • Tag Docker image           │
        │ • Push image to registry     │
        └──────────────┬───────────────┘
                       │ Success
                       ▼
        
        ┌──────────────────────────────┐
        │ STAGE 3: DEPLOY TO STAGING   │
        ├──────────────────────────────┤
        │ • Pull Docker image          │
        │ • Deploy to staging server   │
        │ • Run health checks          │
        └──────────────┬───────────────┘
                       │
                       ▼
        
              ┌─────────────────┐
              │ Staging Server  │
              │ Deployment Done │
              └─────────────────┘

Task 5: Explore in the Wild

1) Open any popular open-source repo on GitHub (Kubernetes, React, FastAPI — pick one you know)
        
        I picked FastAPI

2) Find their .github/workflows/ folder
        
        23 yml files

3) Open one workflow YAML file

        `add-to-project.yml`

4) Write in your notes:
- What triggers it?

        pull_requests_target and issues in that types: opened and reopened

- How many jobs does it have?
        
        one called add-to-project

- What does it do? (best guess)

        uses github actions/add-to-project
        runs when Issues or Pull Requests are opened or labeled in your repository
        supports adding Issues to your project which are transferred into your repository.
