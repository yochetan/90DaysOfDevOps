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

