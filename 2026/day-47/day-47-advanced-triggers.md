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

* Print the PR title: ${{ github.event.pull_request.title }}

* Print the PR author: ${{ github.event.pull_request.user.login }}

* Print the source branch and target branch

3) Add a conditional step that only runs when the PR is merged (closed + merged = true)

Test it: create a PR, push an update to it, then merge it. Watch the workflow fire each time with a different event type.
