Task 1: Understand workflow_call

Before writing any code, research and answer in your notes:

1) What is a reusable workflow?

        A reusable workflow is a GitHub Actions workflow that is designed to be called by other workflows instead of running directly for every event. It lets you write common CI/CD logic once and reuse it across multiple repositories or workflows.

Benefits:

        - Reduces duplicate YAML code.
        - Makes workflows easier to maintain.
        - Ensures consistent CI/CD processes across projects.

Example use cases:

        - Standard build and test pipeline
        - Docker image build and push
        - Security scanning
        - Deployment workflow

2) What is the workflow_call trigger?

        workflow_call is a special GitHub Actions event that allows one workflow to be invoked by another workflow.

Example:

        on:
          workflow_call:
            inputs:
              environment:
                required: true
                type: string

The calling workflow can then pass inputs:

        jobs:
          deploy:
            uses: ./.github/workflows/deploy.yml
            with:
              environment: production

        The reusable workflow starts only when another workflow calls it.

3) How is calling a reusable workflow different from using a regular action (uses:)?

| Reusable Workflow                         | Regular Action                                           |
|-------------------------------------------|----------------------------------------------------------|
| Runs an entire workflow                   | Runs a single action or task                             |
| Can contain multiple jobs                 | Usually performs one specific task                       |
| Invoked at the job level                  | Invoked at the step level                                |
| Can define inputs, secrets, and outputs   | Can also define inputs/outputs, but only for that action |
| Best for reusing complete CI/CD pipelines | Best for reusable individual operations                  |


Reusable workflow:

        jobs:
          build:
            uses: ./.github/workflows/build.yml

Regular action:

        steps:
          - uses: actions/checkout@v4

Think of it this way:

        - Action = reusable function
        - Reusable workflow = reusable program made of multiple functions

4) Where must a reusable workflow file live?

A reusable workflow must be stored in the repository's:

        .github/workflows/

For example:

        my-repo/
        └── .github/
            └── workflows/
                ├── ci.yml
                ├── deploy.yml
                └── reusable-build.yml

It is then referenced like this:

        jobs:
          call-build:
            uses: ./.github/workflows/reusable-build.yml

If you're calling a reusable workflow from another repository:

        jobs:
          call-build:
            uses: owner/repository/.github/workflows/reusable-build.yml@main


Task 2: Create Your First Reusable Workflow

Create .github/workflows/reusable-build.yml:

1) Set the trigger to workflow_call

        on: 
                  workflow_call:

2) Add an inputs: section with:

* app_name (string, required)

        inputs:
                      app_name:
                        description: "Application name"
                        required: true
                        type: string

* environment (string, required, default: staging)

        environment:
                        description: "Deployment environment"
                        default: staging
                        required: true
                        type: string

3) Add a secrets: section with:

* docker_token (required)

        secrets:
                      docker_token: 
                        required: true

4) Create a job that:

* Checks out the code

        - name: Checkout the code
                        uses: actions/checkout@v4

* Prints Building <app_name> for <environment>

        - name: Print build information
                        run: |
                          echo "Building ${{ inputs.app_name }} for ${{ inputs.environment }}"

* Prints Docker token is set: true (never print the actual secret)

        - name: Check Docker token
                        run: |
                          echo "Docker token is set: ${{ secrets.docker_token != '' }}"

`reusable-build.yml`

        name: Reusable Build
        
        on: 
          workflow_call:
            inputs:
              app_name:
                description: "Application name"
                required: true
                type: string
        
              environment:
                description: "Deployment environment"
                default: staging
                required: true
                type: string
        
            secrets:
              docker_token: 
                required: true
              
        
        jobs:
          build:
            runs-on: ubuntu-latest
        
            steps:
              - name: Checkout the code
                uses: actions/checkout@v4
        
              - name: Print build information
                run: |
                  echo "Building ${{ inputs.app_name }} for ${{ inputs.environment }}"
        
              - name: Check Docker token
                run: |
                  echo "Docker token is set: ${{ secrets.docker_token != '' }}"

Verify: This file alone won't run — it needs a caller. That's next.


Task 3: Create a Caller Workflow

Create .github/workflows/call-build.yml:

1) Trigger on push to main

        on: 
          push:
            branches:
              - main

2) Add a job that uses your reusable workflow:

        jobs:
          build:
            uses: ./.github/workflows/reusable-build.yml
            with:
              app_name: "my-web-app"
              environment: "production"
            secrets:
              docker_token: ${{ secrets.DOCKERHUB_TOKEN }}

3) Push to main and watch it run

`call-build.yml`

        name: Call Build
        
        on: 
          push:
            branches:
              - main
        
        jobs:
                  build:
                    uses: ./.github/workflows/reusable-build.yml
                    with:
                      app_name: "my-web-app"
                      environment: "production"
                    secrets:
                      docker_token: ${{ secrets.DOCKERHUB_TOKEN }}

Verify: In the Actions tab, do you see the caller triggering the reusable workflow? Click into the job — can you see the inputs printed?


Task 4: Add Outputs to the Reusable Workflow

Extend reusable-build.yml:

1) Add an outputs: section that exposes a build_version value

        outputs:
              build_version:
                description: "Generated build version"
                value: ${{ jobs.build.outputs.build_version }}

2) Inside the job, generate a version string (e.g., v1.0-<short-sha>) and set it as output

        outputs:
              build_version: ${{ steps.version.outputs.build_version }}
        
        steps:
        
              - name: Generate build version
                id: version
                run: |
                  SHORT_SHA=${GITHUB_SHA::7}
                  BUILD_VERSION="v1.0-${SHORT_SHA}"
                  echo "build_version=$BUILD_VERSION" >> "$GITHUB_OUTPUT"

3) In your caller workflow, add a second job that:

* Depends on the build job (needs:)

        print-version:
                    runs-on: ubuntu-latest
                    needs: build

* Reads and prints the build_version output

        steps:
                      - name: Print build version
                        run: |
                          echo "Build Version: ${{ needs.build.outputs.build_version }}"

Verify: Does the second job print the version from the reusable workflow?

        yeah it showed:
        
        Run echo "Build Version: v1.0-a638830"
        Build Version: v1.0-a638830


Task 5: Create a Composite Action

Create a custom composite action in your repo at .github/actions/setup-and-greet/action.yml:

1) Define inputs: name and language (default: en)

        inputs:
          name:
            description: Name of the person
            required: true
        
          language:
            description: Greeting language
            required: false
            default: en

2) Add steps that:

* Print a greeting in the specified language

        - id: greet
              shell: bash
              run: |
                if [ "${{ inputs.language }}" = "en" ]; then
                  echo "Hello, ${{ inputs.name }}!"
                elif [ "${{ inputs.language }}" = "es" ]; then
                  echo "Hola, ${{ inputs.name }}!"
                elif [ "${{ inputs.language }}" = "fr" ]; then
                  echo "Bonjour, ${{ inputs.name }}!"
                else
                  echo "Hello, ${{ inputs.name }}!"
                fi

* Print the current date and runner OS

        echo "Current Date: $(date)"
        echo "Runner OS: $RUNNER_OS"


* Set an output called greeted with value true

        echo "greeted=true" >> "$GITHUB_OUTPUT"

`action.yml`

        name: Setup and Greet
        description: Prints a greeting, current date, and runner OS
        
        inputs:
          name:
            description: Name of the person
            required: true
        
          language:
            description: Greeting language
            required: false
            default: en
        
        outputs:
          greeted:
            description: Indicates greeting completed
            value: ${{ steps.greet.outputs.greeted }}
        
        runs:
          using: composite
        
          steps:
            - id: greet
              shell: bash
              run: |
                if [ "${{ inputs.language }}" = "en" ]; then
                  echo "Hello, ${{ inputs.name }}!"
                elif [ "${{ inputs.language }}" = "es" ]; then
                  echo "Hola, ${{ inputs.name }}!"
                elif [ "${{ inputs.language }}" = "fr" ]; then
                  echo "Bonjour, ${{ inputs.name }}!"
                else
                  echo "Hello, ${{ inputs.name }}!"
                fi
        
                echo "Current Date: $(date)"
                echo "Runner OS: $RUNNER_OS"
        
                echo "greeted=true" >> "$GITHUB_OUTPUT"

3) Use the composite action in a new workflow with uses: ./.github/actions/setup-and-greet

`composite-action.yml`

        name: Composite Action Demo
        
        on:
          push:
            branches:
              - main
        
        jobs:
          demo:
            runs-on: ubuntu-latest
        
            steps:
              - name: Checkout Repository
                uses: actions/checkout@v4
        
              - name: Run Composite Action
                id: greet
                uses: ./.github/actions/setup-and-greet
                with:
                  name: Chetan
                  language: en
        
              - name: Print Output
                run: |
                  echo "Greeting completed: ${{ steps.greet.outputs.greeted }}"

Verify: Does your custom action run and print the greeting?

yeah the output was:

        Prepare all required actions
        Run ./.github/actions/setup-and-greet
        Run if [ "en" = "en" ]; then
        Hello, Chetan!
        Current Date: Fri Jul 24 22:00:42 UTC 2026
        Runner OS: Linux

