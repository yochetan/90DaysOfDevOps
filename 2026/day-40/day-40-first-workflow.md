Task 1: Set Up

1) Create a new public GitHub repository called github-actions-practice

        https://github.com/yochetan/github-actions-zero-to-hero

2) Clone it locally

        DONE

3) Create the folder structure: .github/workflows/

        DONE

Task 2: Hello Workflow

Create .github/workflows/hello.yml with a workflow that:

1) Triggers on every push

        # This is workflow name
        name: hello
        
        # runs on a trigger
        on: 
          push:
            branches: main
        
        # these are jobs inside the workflow
        jobs:
        
          # this is the name of the job
          greet:
            # this is github runner, each job runs on a runner
            runs-on: ubuntu-latest
            steps:
              - name: say hello to the users
                run: echo "hello brev"

2) Has one job called greet

        jobs:
        
          # this is the name of the job
          greet:
            # this is github runner, each job runs on a runner
            runs-on: ubuntu-latest
            steps:
              - name: say hello to the users
                run: echo "hello brev"

3) Runs on ubuntu-latest

            runs-on: ubuntu-latest

4) Has two steps:
- Step 1: Check out the code using actions/checkout

        steps:
              - name: Checkout Repository
                uses: actions/checkout@v4
        
- Step 2: Print Hello from GitHub Actions!

        steps:
              - name: say hello to the users
                run: echo "hello brev"

Push it. Go to the Actions tab on GitHub and watch it run.

Verify: Is it green? Click into the job and read every step.

        yess! 


Task 3: Understand the Anatomy

Look at your workflow file and write in your notes what each key does:


        on:	Defines what event triggers the workflow
        jobs:	Contains the jobs to run
        runs-on:	Specifies the runner machine
        steps:	Lists actions/commands in a job
        uses:	Runs a reusable GitHub Action
        run:	Executes shell commands
        name:	Gives a readable name to a step


Task 4: Add More Steps

Update hello.yml to also:

1) Print the current date and time

2) Print the name of the branch that triggered the run (hint: GitHub provides this as a variable)

3) List the files in the repo

4) Print the runner's operating system

Push again — watch the new run.

      - name: Print Current Date and Time
          run: date
        
      - name: Print Branch Name
          run: |
            echo "Branch: ${{ github.ref_name }}"
        
      - name: List Repository Files
          run: ls -la
        
      - name: Print Runner Operating System
          run: |
            echo "Runner OS: $RUNNER_OS"


Task 5: Break It On Purpose

1) Add a step that runs a command that will fail (e.g., exit 1 or a misspelled command)

        - name: Intentional Failure
          run: this_command_does_not_exist
        
        Push the change:
        
        git add .
        git commit -m "Break pipeline for testing"
        git push origin main

2) Push and observe what happens in the Actions tab

        In the Actions tab you will see:
        
        ❌ Workflow failed
        
        The failed job will be marked with a red X. Clicking on the job shows which step failed and the error message.
        
        Run this_command_does_not_exist
        this_command_does_not_exist: command not found
        Error: Process completed with exit code 127.

3) Fix it and push again

        Remove the failing step or replace it with a valid command:
        
        - name: Fixed Step
          run: echo "Pipeline fixed!"
        
        Push again:
        
        git add .
        git commit -m "Fix pipeline"
        git push origin main
        
        The workflow should now show:
        
        ✅ Workflow completed successfully

Write in your notes: What does a failed pipeline look like? How do you read the error?

What does a failed pipeline look like?

        - The workflow run is marked with a red X.
        - The failed job and step are highlighted.
        - Subsequent steps usually do not run after the failure.
        - GitHub displays the error message and exit code in the logs.

How do you read the error?

        - Open the failed workflow run.
        - Click the failed job.
        - Find the step marked with a red X.
        - Read the logs from top to bottom.
        - Look for:
        * Error messages
        * Failed commands
        * Exit codes (exit code 1, 127, etc.)
        - Fix the issue and push the changes again.

Key takeaway: A CI/CD pipeline is only as useful as your ability to read its logs. Most debugging starts by identifying the exact step that failed and examining its error message.
