Task 1: Trigger on Pull Request

1) Create .github/workflows/pr-check.yml

        done

**pr-check.yml

        name: Trigger on pull request
        
        on:
           pull_request:
              branches: 
               - main
              types:
                - opened
                - synchronize
        
        jobs:
            pr-check:
                runs-on: ubuntu-latest
        
                steps:
        
                    - name: Check Repository
                      uses: actions/checkout@v4
                    
                    - name: PR check running for branch
                      run: |
                         echo "PR check running for branch: ${{ github.head_ref }}"

2) Trigger it only when a pull request is opened or updated against main

        on:
                   pull_request:
                      branches: 
                       - main
                      types:
                        - opened
                        - synchronize

3) Add a step that prints: PR check running for branch: <branch name>

        - name: PR check running for branch
                              run: |
                                 echo "PR check running for branch: ${{ github.head_ref }}"

4) Create a new branch, push a commit, and open a PR

        yeah, did

5) Watch the workflow run automatically

        yes

Verify: Does it show up on the PR page?

        https://github.com/yochetan/github-actions-zero-to-hero/actions/runs/29368309409/job/87205225046?pr=1


Task 2: Scheduled Trigger

1) Add a schedule: trigger to any workflow using cron syntax
        
        schedule:
            - cron: 0 0 * * *

2) Set it to run every day at midnight UTC

        schedule:
            - cron: 0 0 * * *

        0 → Minute 0
        0 → Hour 0 (midnight)
        * → Every day of the month
        * → Every month
        * → Every day of the week

3) Write in your notes: What is the cron expression for every Monday at 9 AM?

        0 9 * * 1


Task 3: Manual Trigger

1) Create .github/workflows/manual.yml with a workflow_dispatch: trigger

**manual.yml

        name: Manual Workflow
        
        on:
          workflow_dispatch:
            inputs:
              environment:
                description: "Select the deployment environment"
                required: true
                default: "staging"
                type: choice
                options:
                  - staging
                  - production
        
        jobs:
          manual-job:
            runs-on: ubuntu-latest
        
            steps:
              - name: Print selected environment
                run: |
                  echo "Selected environment: ${{ inputs.environment }}"

2) Add an input that asks for an environment name (staging/production)

        on:
                  workflow_dispatch:
                    inputs:
                      environment:
                        description: "Select the deployment environment"
                        required: true
                        default: "staging"
                        type: choice
                        options:
                          - staging
                          - production

3) Print the input value in a step

        steps:
                      - name: Print selected environment
                        run: |
                          echo "Selected environment: ${{ inputs.environment }}"

4) Go to the Actions tab → find the workflow → click Run workflow

        yeah, did.

Verify: Can you trigger it manually and see your input printed?

        yes, we can trigger manually as it is on workflow_dispatch and we can also see the input that is printed.


Task 4: Matrix Builds
Create .github/workflows/matrix.yml that:

1) Uses a matrix strategy to run the same job across:
* Python versions: 3.10, 3.11, 3.12

**matrix.yml

        name: Matrix Build
        
        on: 
          workflow_dispatch:
        
        jobs:
          test:
            runs-on: ubuntu-latest
        
            strategy:
              matrix:
                python-version: ["3.10", "3.11", "3.12"]
        
            steps:
             - name: Checkout respository
               uses: actions/checkout@v4
        
             - name: Set up python
               uses: actions/setup-python@v5
               with:
                 python-version: ${{ matrix.python-version }}
        
             - name: Print python version
               run: python --version 

2) Each job installs Python and prints the version

           - name: Set up python
             uses: actions/setup-python@v5
             with:
                python-version: ${{ matrix.python-version }}
                
           - name: Print python version
             run: python --version 

3) Watch all 3 run in parallel

        yeah they did.

Then extend the matrix to also include 2 operating systems — how many total jobs run now?

        Total jobs:
        
        2 operating systems
        3 Python versions
        
        Calculation:
        
        2 × 3 = 6 jobs


Task 5: Exclude & Fail-Fast

1) In your matrix, exclude one specific combination (e.g., Python 3.10 on Windows)

        name: Matrix Build
        
        on: 
          workflow_dispatch:
        
        jobs:
          test:
            runs-on: ubuntu-latest
        
            strategy:
              matrix:
                os:
                 - ubuntu-latest
                python-version: ["3.10", "3.11", "3.12"]
        
                exclude:
                 - os: ubuntu-latest
                   python-version: "3.11"
        
            steps:
             - name: Checkout respository
               uses: actions/checkout@v4
        
             - name: Set up python
               uses: actions/setup-python@v5
               with:
                 python-version: ${{ matrix.python-version }}
        
             - name: Print python version
               run: python --version
      

2) Set fail-fast: false — trigger a failure in one job and observe what happens to the rest

        name: Matrix Build
        
        on: 
          workflow_dispatch:
        
        jobs:
          test:
            runs-on: ubuntu-latest
        
            strategy:
              fail-fast: false
              matrix:
                os:
                 - ubuntu-latest
                python-version: ["3.10", "3.11", "3.12"]
        
                exclude:
                 - os: ubuntu-latest
                   python-version: "3.11"
        
            steps:
             - name: Checkout respository
               uses: actions/checkout@v4
        
             - name: Set up python
               uses: actions/setup-python@v5
               with:
                 python-version: ${{ matrix.python-version }}
        
             - name: Print python version
               run: python --version

3) Write in your notes: What does fail-fast: true (the default) do vs false?

* fail-fast: true (default):

        If one matrix job fails, GitHub cancels the remaining in-progress or queued matrix jobs to save time and resources.

* fail-fast: false:

        If one matrix job fails, all other matrix jobs continue running until they finish. This is useful when you want to see results for every OS/version combination in a single run.
