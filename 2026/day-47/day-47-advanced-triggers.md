Task 1: Pull Request Event Types

Create .github/workflows/pr-lifecycle.yml that triggers on pull_request with specific activity types:

1) Trigger on: opened, synchronize, reopened, closed

        on: 
          pull_request:
            types: 
             - opened
             - closed
             - synchronize
             - reopened

2) Add steps that:

* Print which event type fired: ${{ github.event.action }}

        - name: Print Event Type
                run: |
                  echo "Event Type: ${{ github.event.action }}"

* Print the PR title: ${{ github.event.pull_request.title }}

        - name: Print PR Title
                run: | 
                  echo "PR Title: ${{ github.event.pull_request.title }}"

* Print the PR author: ${{ github.event.pull_request.user.login }}

        - name: Print PR Author
                run: | 
                  echo "PR Author: ${{ github.event.pull_request.user.login }}"

* Print the source branch and target branch

        - name: Print Source and Target Branches
                run: |
                  echo "Source Branch: ${{ github.head_ref }}"
                  echo "Target Branch: ${{ github.base_ref }}"

3) Add a conditional step that only runs when the PR is merged (closed + merged = true)

        - name: Run Only When PR Is Merged
                if: github.event.action == 'closed' && github.event.pull_request.merged == true
                run: |
                  echo "Pull Request has been merged!"
                  echo "Merged by: ${{ github.event.pull_request.merged_by.login }}"

Test it: create a PR, push an update to it, then merge it. Watch the workflow fire each time with a different event type.


`pr-lifecycle.yml`

        name: Pull request event types
        
        on: 
          pull_request:
            types: 
             - opened
             - closed
             - synchronize
             - reopened
        
        jobs:
          pr-lifecycle:
            runs-on: ubuntu-latest
        
            steps:
              - name: Print Event Type
                run: |
                  echo "Event Type: ${{ github.event.action }}"
        
              - name: Print PR Title
                run: | 
                  echo "PR Title: ${{ github.event.pull_request.title }}"
        
              - name: Print PR Author
                run: | 
                  echo "PR Author: ${{ github.event.pull_request.user.login }}"
        
              - name: Print Source and Target Branches
                run: |
                  echo "Source Branch: ${{ github.head_ref }}"
                  echo "Target Branch: ${{ github.base_ref }}"
        
              - name: Run Only When PR Is Merged
                if: github.event.action == 'closed' && github.event.pull_request.merged == true
                run: |
                  echo "Pull Request has been merged!"
                  echo "Merged by: ${{ github.event.pull_request.merged_by.login }}"

Opened:

        Run echo "Event Type: opened"
        Event Type: opened
        0s
        Run echo "PR Title: Add PR lifecycle workflow to GitHub Actions"
        PR Title: Add PR lifecycle workflow to GitHub Actions
        0s
        Run echo "PR Author: yochetan"
        PR Author: yochetan
        0s
        Run echo "Source Branch: yochetan-patch-1"
        Source Branch: yochetan-patch-1
        Target Branch: main

Synchronize:

        Run echo "Event Type: synchronize"
        Event Type: synchronize
        0s
        Run echo "PR Title: Add PR lifecycle workflow to GitHub Actions"
        PR Title: Add PR lifecycle workflow to GitHub Actions
        0s
        Run echo "PR Author: yochetan"
        PR Author: yochetan
        0s
        Run echo "Source Branch: yochetan-patch-1"
        Source Branch: yochetan-patch-1
        Target Branch: main

Closed:

        Run echo "Event Type: closed"
        Event Type: closed
        0s
        Run echo "PR Title: Add PR lifecycle workflow to GitHub Actions"
        PR Title: Add PR lifecycle workflow to GitHub Actions
        0s
        Run echo "PR Author: yochetan"
        PR Author: yochetan
        0s
        Run echo "Source Branch: yochetan-patch-1"
        Source Branch: yochetan-patch-1
        Target Branch: main

Reopened:

        Run echo "Event Type: reopened"
        Event Type: reopened
        0s
        Run echo "PR Title: Add PR lifecycle workflow to GitHub Actions"
        PR Title: Add PR lifecycle workflow to GitHub Actions
        0s
        Run echo "PR Author: yochetan"
        PR Author: yochetan
        0s
        Run echo "Source Branch: yochetan-patch-1"
        Source Branch: yochetan-patch-1
        Target Branch: main

When PR is merged:

        Run echo "Event Type: closed"
        Event Type: closed
        0s
        Run echo "PR Title: Add PR lifecycle workflow to GitHub Actions"
        PR Title: Add PR lifecycle workflow to GitHub Actions
        0s
        Run echo "PR Author: yochetan"
        PR Author: yochetan
        0s
        Run echo "Source Branch: yochetan-patch-1"
        Source Branch: yochetan-patch-1
        Target Branch: main
        0s
        Run echo "Pull Request has been merged!"
        Pull Request has been merged!
        Merged by: yochetan


Task 2: PR Validation Workflow

Create .github/workflows/pr-checks.yml — a real-world PR gate:

1) Trigger on pull_request to main

        on:
          pull_request:
            branches:
              - main

2) Add a job file-size-check that:

* Checks out the code

        - name: Checkout repository
                uses: actions/checkout@v4

* Fails if any file in the PR is larger than 1 MB

        - name: Check file sizes
                run: |
                  LIMIT=$((1024 * 1024)) # 1 MB
        
                  for file in $(find . -type f); do
                    size=$(stat -c%s "$file")
        
                    if [ "$size" -gt "$LIMIT" ]; then
                      echo "File '$file' is larger than 1 MB."
                      exit 1
                    fi
                  done
        
                  echo "All files are under 1 MB."

3) Add a job branch-name-check that:

* Reads the branch name from ${{ github.head_ref }}

        - name: Validate branch name
                run: |
                  BRANCH="${{ github.head_ref }}"
        
                  echo "Branch: $BRANCH"

* Fails if it doesn't follow the pattern feature/*, fix/*, or docs/*

        if [[ "$BRANCH" =~ ^(feature|fix|docs)/.+$ ]]; then
                    echo "Branch name is valid."
                  else
                    echo "Invalid branch name."
                    echo "Branch must start with feature/, fix/, or docs/"
                    exit 1
                  fi

4) Add a job pr-body-check that:

* Reads the PR body: ${{ github.event.pull_request.body }}

        - name: Check PR description
                run: |
                  BODY="${{ github.event.pull_request.body }}"

* Warns (but doesn't fail) if the PR description is empty

        if [ -z "$BODY" ]; then
                    echo "::warning::PR description is empty."
                  else
                    echo "PR description found."
                  fi

Verify: Open a PR from a badly named branch — does the check fail?

`pr-checks.yml`

        name: PR Checks
        
        on:
          pull_request:
            branches:
              - main
        
        jobs:
          file-size-check:
            runs-on: ubuntu-latest
        
            steps:
              - name: Checkout repository
                uses: actions/checkout@v4
        
              - name: Check file sizes
                run: |
                  LIMIT=$((1024 * 1024)) # 1 MB
        
                  for file in $(find . -type f); do
                    size=$(stat -c%s "$file")
        
                    if [ "$size" -gt "$LIMIT" ]; then
                      echo "File '$file' is larger than 1 MB."
                      exit 1
                    fi
                  done
        
                  echo "All files are under 1 MB."
        
          branch-name-check:
            runs-on: ubuntu-latest
        
            steps:
              - name: Validate branch name
                run: |
                  BRANCH="${{ github.head_ref }}"
        
                  echo "Branch: $BRANCH"
        
                  if [[ "$BRANCH" =~ ^(feature|fix|docs)/.+$ ]]; then
                    echo "Branch name is valid."
                  else
                    echo "Invalid branch name."
                    echo "Branch must start with feature/, fix/, or docs/"
                    exit 1
                  fi
        
          pr-body-check:
            runs-on: ubuntu-latest
        
            steps:
              - name: Check PR description
                run: |
                  BODY="${{ github.event.pull_request.body }}"
        
                  if [ -z "$BODY" ]; then
                    echo "::warning::PR description is empty."
                  else
                    echo "PR description found."
                  fi

Outputs:

mybranch

file-size-check

        All files are under 1 MB.

branch-name-check

        Branch: mybranch
        Invalid branch name.
        Branch must start with feature/, fix/, or docs/
        Error: Process completed with exit code 1.

pr-body-check

        Warning: PR description is empty.

feature/login-page

file-size-check

        All files are under 1 MB.

branch-name-check

        Branch: feature/login-page
        Branch name is valid.

pr-body-check

        PR description found.


Task 3: Scheduled Workflows (Cron Deep Dive)

Create .github/workflows/scheduled-tasks.yml:

1) Add a schedule trigger with cron: '30 2 * * 1' (every Monday at 2:30 AM UTC)

        schedule:
            - cron: '30 2 * * 1'

2) Add another cron entry: '0 */6 * * *' (every 6 hours)

        - cron: '0 */6 * * *'

3) In the job, print which schedule triggered using ${{ github.event.schedule }}

        - name: Print cron schedule
                run: |
                  if [ "${{ github.event_name }}" = "schedule" ]; then
                    echo "Cron that triggered this run: ${{ github.event.schedule }}"
                  else
                    echo "This workflow was triggered manually."
                  fi

4) Add a step that acts as a health check — curl a URL and check the response code

        - name: Health Check
                run: |
                  URL="https://www.google.com"
        
                  STATUS_CODE=$(curl -o /dev/null -s -w "%{http_code}" "$URL")
        
                  echo "Response Code: $STATUS_CODE"
        
                  if [ "$STATUS_CODE" -eq 200 ]; then
                    echo "Health check passed."
                  else
                    echo "Health check failed."
                    exit 1
                  fi

Write in your notes:

* The cron expression for: every weekday at 9 AM IST

        30 3 * * 1-5

* The cron expression for: first day of every month at midnight

        0 0 1 * *

* Why GitHub says scheduled workflows may be delayed or skipped on inactive repos

        GitHub warns that scheduled workflows are best effort rather than guaranteed. They may be delayed or skipped because:
        
        - Scheduled events share infrastructure with many repositories, so runs can be delayed during periods of high load.
        - Public repositories with no recent activity may have scheduled workflows automatically disabled after a period of inactivity (currently 60 days).
        - If a repository is inactive, GitHub may stop triggering scheduled workflows until there is new activity, such as a push or a manual re-enable.

Important: Also add workflow_dispatch so you can test it manually without waiting for the schedule.

`scheduled-tasks.yml`

        name: Scheduled Tasks
        
        on:
          schedule:
            # Every Monday at 2:30 AM UTC
            - cron: '30 2 * * 1'
        
            # Every 6 hours
            - cron: '0 */6 * * *'
        
          workflow_dispatch:
        
        jobs:
          scheduled-job:
            runs-on: ubuntu-latest
        
            steps:
              - name: Print trigger type
                run: |
                  echo "Workflow triggered by: ${{ github.event_name }}"
        
              - name: Print cron schedule
                run: |
                  if [ "${{ github.event_name }}" = "schedule" ]; then
                    echo "Cron that triggered this run: ${{ github.event.schedule }}"
                  else
                    echo "This workflow was triggered manually."
                  fi
        
              - name: Health Check
                run: |
                  URL="https://www.google.com"
        
                  STATUS_CODE=$(curl -o /dev/null -s -w "%{http_code}" "$URL")
        
                  echo "Response Code: $STATUS_CODE"
        
                  if [ "$STATUS_CODE" -eq 200 ]; then
                    echo "Health check passed."
                  else
                    echo "Health check failed."
                    exit 1
                  fi


Task 4: Path & Branch Filters

Create .github/workflows/smart-triggers.yml:

1) Trigger on push but only when files in src/ or app/ change:

        on:
          push:
            paths:
              - 'src/**'
              - 'app/**'

2) Add paths-ignore in a second workflow that skips runs when only docs change:

        paths-ignore:
          - '*.md'
          - 'docs/**'

3) Add branch filters to only trigger on main and release/* branches

        push:
            branches:
              - main
              - 'release/*'

4) Test it: push a change to a .md file — does the workflow skip?

        yeah it does skips.

Write in your notes: When would you use paths vs paths-ignore?

paths	

        - Run the workflow only when specific files or directories change. This saves CI time by ignoring unrelated changes.

paths-ignore

        - Skip the workflow when changes are limited to certain files (such as documentation or Markdown files), but still run it for code changes.

`smart-triggers.yml`

        name: Smart Triggers
        
        on:
          push:
            branches:
              - main
              - 'release/*'
            paths:
              - 'src/**'
              - 'app/**'
        
        jobs:
          build:
            runs-on: ubuntu-latest
        
            steps:
              - uses: actions/checkout@v4
        
              - name: Print trigger information
                run: |
                  echo "Workflow triggered!"
                  echo "Branch: ${{ github.ref_name }}"
                  echo "Commit: ${{ github.sha }}"

`ignore-docs.yml`

        name: Ignore Docs Changes
        
        on:
          push:
            branches:
              - main
              - 'release/*'
            paths-ignore:
              - '*.md'
              - 'docs/**'
        
        jobs:
          test:
            runs-on: ubuntu-latest
        
            steps:
              - uses: actions/checkout@v4
        
              - name: Run workflow
                run: echo "Changes are not documentation-only."


Task 5: workflow_run — Chain Workflows Together

Create two workflows:

1) .github/workflows/tests.yml — runs tests on every push

`tests.yml`

        name: Run Tests
        
        on:
          push:
        
        jobs:
          test:
            runs-on: ubuntu-latest
        
            steps:
              - uses: actions/checkout@v4
        
              - name: Simulate Tests
                run: |
                  echo "Running tests..."
                  echo "All tests passed!"

2) .github/workflows/deploy-after-tests.yml — triggers only after tests.yml completes successfully:

        on:
          workflow_run:
            workflows: ["Run Tests"]
            types: [completed]

3) In the deploy workflow, add a conditional:

* Only proceed if the triggering workflow succeeded (${{ github.event.workflow_run.conclusion == 'success' }})

        - name: Deploy
                if: ${{ github.event.workflow_run.conclusion == 'success' }}
                run: |
                  echo "Tests passed."
                  echo "Deploying application..."

* Print a warning and exit if it failed

        - name: Stop if Tests Failed
                if: ${{ github.event.workflow_run.conclusion != 'success' }}
                run: |
                  echo "Tests failed. Deployment cancelled."
                  exit 1

`deploy-after-tests.yml`

        name: Deploy After Tests
        
        on:
          workflow_run:
            workflows: ["Run Tests"]
            types:
              - completed
        
        jobs:
          deploy:
            runs-on: ubuntu-latest
        
            steps:
              - name: Check Test Result
                run: |
                  echo "Workflow conclusion: ${{ github.event.workflow_run.conclusion }}"
        
              - name: Stop if Tests Failed
                if: ${{ github.event.workflow_run.conclusion != 'success' }}
                run: |
                  echo "Tests failed. Deployment cancelled."
                  exit 1
        
              - name: Deploy
                if: ${{ github.event.workflow_run.conclusion == 'success' }}
                run: |
                  echo "Tests passed."
                  echo "Deploying application..."

Verify: Push a commit — does the test workflow run first, then trigger the deploy workflow?

        yes the tests workflow runs first, then after the completion the deploy-after-tests workflows runs


Task 6: repository_dispatch — External Event Triggers

1) Create .github/workflows/external-trigger.yml with trigger repository_dispatch

        on:
          repository_dispatch:

2) Set it to respond to event type: deploy-request

        on:
          repository_dispatch:
            types:
              - deploy-request

3) Print the client payload: ${{ github.event.client_payload.environment }}

        - name: Print environment
                run: echo "Environment = ${{ github.event.client_payload.environment }}"

`external-trigger.yml`

        name: External Trigger
        
        on:
          repository_dispatch:
            types:
              - deploy-request
        
        jobs:
          deploy:
            runs-on: ubuntu-latest
        
            steps:
              - name: Print event type
                run: echo "Repository dispatch received"
        
              - name: Print environment
                run: echo "Environment = ${{ github.event.client_payload.environment }}"
        
              - name: Print full payload
                run: echo '${{ toJson(github.event.client_payload) }}'

4) Trigger it using curl or gh:

        gh api repos/<owner>/<repo>/dispatches \
          -f event_type=deploy-request \
          -f client_payload='{"environment":"production"}'

Write in your notes: When would an external system (like a Slack bot or monitoring tool) trigger a pipeline?

        An external system triggers a pipeline to automate actions based on events.

        Examples:

        - Slack bot: Runs a deployment when someone types /deploy.
        - Monitoring tool: Starts a recovery workflow when a server fails.
        - Another CI/CD system: Triggers the next stage after a build completes.

Output:
        
        Run echo "Repository dispatch received"
        Repository dispatch received
        0s
        Run echo "Environment = production"
        Environment = production
        0s
        Run echo '{
        {
          "environment": "production"
        }
