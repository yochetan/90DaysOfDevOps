Task 1: Set Up the Project Repo

1) Create a new repo called github-actions-capstone (or use your existing github-actions-practice)

        done.

2) Add a simple app — pick any one:

* A Python Flask/FastAPI app with one endpoint

* A Node.js Express app with one endpoint

`app.js`

        const express = require("express");
        
        const app = express();
        
        const PORT = process.env.PORT || 3000;
        
        app.get("/", (req, res) => {
          res.send("GitHub Actions Capstone");
        });
        
        app.get("/health", (req, res) => {
          res.status(200).json({
            status: "healthy"
          });
        });
        
        app.listen(PORT, () => {
          console.log(`Server running on port ${PORT}`);
        });

* Your Dockerized app from Day 36

3) Add a Dockerfile and a basic test (even a script that curls the health endpoint counts)

`Dockerfile`

        FROM node:22-alpine
        
        WORKDIR /app
        
        COPY package*.json ./
        
        RUN npm install
        
        COPY . .

        RUN chmod +x test.sh
        
        EXPOSE 3000
        
        CMD ["npm", "start"]

`test.sh`

        #!/bin/bash
        
        echo "Waiting for application..."
        
        sleep 5
        
        curl -f http://localhost:3000/health
        
        echo
        echo "Health check passed!"

4) Add a README.md with a project description

        # GitHub Actions Capstone
        
        A simple Node.js Express application built for practicing GitHub Actions CI/CD workflows.
        
        ## Features
        
        - Express web server
        - Health endpoint
        - Docker support
        - Simple health check test
        - Ready for GitHub Actions pipelines
        
        ## Endpoints
        
        | Endpoint | Description |
        |----------|-------------|
        | / | Home page |
        | /health | Health check |
        
        ## Run Locally
        
        Install dependencies:
        
        ```bash
        npm install
        ```
        
        Start the application:
        
        ```bash
        npm start
        ```
        
        Visit:
        
        ```
        http://localhost:3000
        ```
        
        Health check:
        
        ```
        http://localhost:3000/health
        ```
        
        ## Docker
        
        Build:
        
        ```bash
        docker build -t github-actions-capstone .
        ```
        
        Run:
        
        ```bash
        docker run -p 3000:3000 github-actions-capstone
        ```
        
        ## Test
        
        ```bash
        ./test.sh
        ```# GitHub Actions Capstone
        
        A simple Node.js Express application built for practicing GitHub Actions CI/CD workflows.
        
        ## Features
        
        - Express web server
        - Health endpoint
        - Docker support
        - Simple health check test
        - Ready for GitHub Actions pipelines
        
        ## Endpoints
        
        | Endpoint | Description |
        |----------|-------------|
        | / | Home page |
        | /health | Health check |
        
        ## Run Locally
        
        Install dependencies:
        
        ```bash
        npm install
        ```
        
        Start the application:
        
        ```bash
        npm start
        ```
        
        Visit:
        
        ```
        http://localhost:3000
        ```
        
        Health check:
        
        ```
        http://localhost:3000/health
        ```
        
        ## Docker
        
        Build:
        
        ```bash
        docker build -t github-actions-capstone .
        ```
        
        Run:
        
        ```bash
        docker run -p 3000:3000 github-actions-capstone
        ```
        
        ## Test
        
        ```bash
        ./test.sh
        ```


Task 2: Reusable Workflow — Build & Test

Create .github/workflows/reusable-build-test.yml:

1) Trigger: workflow_call

        on:
          workflow_call:

2) Inputs: python_version (or node_version), run_tests (boolean, default: true)

        inputs:
              node_version:
                description: "Node.js version"
                required: true
                type: string
              run_tests:
                description: "Run tests"
                required: false
                default: true
                type: boolean


3) Steps:

* Check out code

        - name: Checkout repository
                uses: actions/checkout@v4

* Set up the language runtime

        - name: Setup Node.js
                uses: actions/setup-node@v4
                with:
                  node-version: ${{ inputs.node_version }}

* Install dependencies

        - name: Install dependencies
                run: npm install


* Run tests (only if run_tests is true)
        
        - name: Run tests
                if: ${{ inputs.run_tests }}
                id: tests
                run: sh test.sh
        - name: Set test result - Passed
                if: ${{ success() && inputs.run_tests }}
                id: test-result
                run: echo "result=passed" >> "$GITHUB_OUTPUT"

* Set output: test_result with value passed or failed

      - name: Set test result - Passed
        if: ${{ success() && inputs.run_tests }}
        id: test-result
        run: echo "result=passed" >> "$GITHUB_OUTPUT"

      - name: Set test result - Failed
        if: ${{ failure() && inputs.run_tests }}
        id: test-result-failed
        run: echo "result=failed" >> "$GITHUB_OUTPUT"

This workflow does NOT deploy — it only builds and tests.

`reusable-build-test.yml`

        name: Reusable Build & Test
        
        on:
          workflow_call:
            inputs:
              node_version:
                description: "Node.js version"
                required: true
                type: string
              run_tests:
                description: "Run tests"
                required: false
                default: true
                type: boolean
        
            outputs:
              test_result:
                description: "Result of the test step"
                value: ${{ jobs.build-test.outputs.test_result }}
        
        jobs:
          build-test:
            runs-on: ubuntu-latest
        
            outputs:
              test_result: ${{ steps.test-result.outputs.result }}
        
            steps:
              - name: Checkout repository
                uses: actions/checkout@v4
        
              - name: Setup Node.js
                uses: actions/setup-node@v4
                with:
                  node-version: ${{ inputs.node_version }}
        
              - name: Install dependencies
                run: npm install
        
              - name: Start application
                if: ${{ inputs.run_tests }}
                run: |
                  npm start &
                  sleep 5
        
              - name: Run tests
                if: ${{ inputs.run_tests }}
                id: tests
                run: sh test.sh
        
              - name: Set test result - Passed
                if: ${{ success() && inputs.run_tests }}
                id: test-result
                run: echo "result=passed" >> "$GITHUB_OUTPUT"
        
              - name: Set test result - Failed
                if: ${{ failure() && inputs.run_tests }}
                id: test-result-failed
                run: echo "result=failed" >> "$GITHUB_OUTPUT"
        
              - name: Set test result - Skipped
                if: ${{ !inputs.run_tests }}
                id: test-result
                run: echo "result=skipped" >> "$GITHUB_OUTPUT"


Task 3: Reusable Workflow — Docker Build & Push

Create .github/workflows/reusable-docker.yml:

1) Trigger: workflow_call

        on:
          workflow_call:

2) Inputs: image_name (string), tag (string)

            inputs:
              image_name:
                description: "Docker image name"
                required: true
                type: string
        
              tag:
                description: "Docker image tag"
                required: true
                type: string


3) Secrets: docker_username, docker_token

            secrets:
              docker_username:
                description: "Docker Hub username"
                required: true
        
              docker_token:
                description: "Docker Hub access token"
                required: true

4) Steps:

* Check out code

      - name: Checkout repository
        uses: actions/checkout@v4

* Log in to Docker Hub

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.docker_username }}
          password: ${{ secrets.docker_token }}

* Build and push the image with the given tag

      - name: Build and push Docker image
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: ${{ secrets.docker_username }}/${{ inputs.image_name }}:${{ inputs.tag }}

* Set output: image_url with the full image path

      - name: Set image URL output
        id: image-url
        run: |
          echo "image_url=${{ secrets.docker_username }}/${{ inputs.image_name }}:${{ inputs.tag }}" >> "$GITHUB_OUTPUT"

`reusable-docker.yml`

        name: Reusable Docker Build & Push
        
        on:
          workflow_call:
            inputs:
              image_name:
                description: "Docker image name"
                required: true
                type: string
        
              tag:
                description: "Docker image tag"
                required: true
                type: string
        
            secrets:
              docker_username:
                description: "Docker Hub username"
                required: true
        
              docker_token:
                description: "Docker Hub access token"
                required: true
        
            outputs:
              image_url:
                description: "Full Docker image URL"
                value: ${{ jobs.docker.outputs.image_url }}
        
        jobs:
          docker:
            runs-on: ubuntu-latest
        
            outputs:
              image_url: ${{ steps.image-url.outputs.image_url }}
        
            steps:
              - name: Checkout repository
                uses: actions/checkout@v4
        
              - name: Log in to Docker Hub
                uses: docker/login-action@v3
                with:
                  username: ${{ secrets.docker_username }}
                  password: ${{ secrets.docker_token }}
        
              - name: Build and push Docker image
                uses: docker/build-push-action@v6
                with:
                  context: .
                  push: true
                  tags: ${{ secrets.docker_username }}/${{ inputs.image_name }}:${{ inputs.tag }}
        
              - name: Set image URL output
                id: image-url
                run: |
                  echo "image_url=${{ secrets.docker_username }}/${{ inputs.image_name }}:${{ inputs.tag }}" >> "$GITHUB_OUTPUT"


Task 4: PR Pipeline

Create .github/workflows/pr-pipeline.yml:

1) Trigger: pull_request to main (types: opened, synchronize)

        on:
          pull_request:
            branches:
              - main
            types:
              - opened
              - synchronize

2) Call the reusable build-test workflow:

* Run tests: true

        build-test:
            uses: ./.github/workflows/reusable-build-test.yml
            with:
              node_version: "22"
              run_tests: true

3) Add a standalone job pr-comment that:

* Runs after the build-test job

        pr-comment:
            runs-on: ubuntu-latest
            needs: build-test

* Prints a summary: "PR checks passed for branch: <branch>"

      - name: Print PR summary
        run: |
          echo "PR checks passed for branch: ${{ github.head_ref }}"
          echo "Test Result: ${{ needs.build-test.outputs.test_result }}"

4) Do NOT build or push Docker images on PRs

        yeah

Verify: Open a PR — does it run tests only (no Docker push)?

        yes it runs only tests

`pr-pipeline.yml`

        name: PR Pipeline
        
        on:
          pull_request:
            branches:
              - main
            types:
              - opened
              - synchronize
        
        jobs:
          build-test:
            uses: ./.github/workflows/reusable-build-test.yml
            with:
              node_version: "22"
              run_tests: true
        
          pr-comment:
            runs-on: ubuntu-latest
            needs: build-test
        
            steps:
              - name: Print PR summary
                run: |
                  echo "PR checks passed for branch: ${{ github.head_ref }}"
                  echo "Test Result: ${{ needs.build-test.outputs.test_result }}"


Task 5: Main Branch Pipeline

Create .github/workflows/main-pipeline.yml:

1) Trigger: push to main

        on:
          push:
            branches:
              - main

2) Job 1: Call the reusable build-test workflow

        build-test:
            uses: ./.github/workflows/reusable-build-test.yml
            with:
              node_version: "22"
              run_tests: true

3) Job 2 (depends on Job 1): Call the reusable Docker workflow

* Tag: latest and sha-<short-commit-hash>

        docker:
            needs: build-test
        
            uses: ./.github/workflows/reusable-docker.yml
            with:
              image_name: github-actions-capstone
              tag: latest
            secrets:
              docker_username: ${{ secrets.DOCKER_USERNAME }}
              docker_token: ${{ secrets.DOCKER_TOKEN }}

        docker-sha:
            needs: build-test
        
            uses: ./.github/workflows/reusable-docker.yml
            with:
              image_name: github-actions-capstone
              tag: sha-${{ github.sha }}
            secrets:
              docker_username: ${{ secrets.DOCKER_USERNAME }}
              docker_token: ${{ secrets.DOCKER_TOKEN }}

4) Job 3 (depends on Job 2): deploy job that:

* Prints "Deploying image: <image_url> to production"
* Uses environment: production (set this up in repo Settings → Environments)
* Requires manual approval if you've set up environment protection rules

        deploy:
            needs:
              - docker
              - docker-sha
        
            runs-on: ubuntu-latest
        
            environment:
              name: production
        
            steps:
              - name: Deploy
                run: |
                  echo "Deploying image: ${{ needs.docker.outputs.image_url }} to production"

Verify: Merge a PR to main — does it run tests → build Docker → deploy in sequence?

        yes it did in the above sequence

`main-pipeline.yml`

        name: Main Pipeline
        
        on:
          push:
            branches:
              - main
        
        jobs:
          build-test:
            uses: ./.github/workflows/reusable-build-test.yml
            with:
              node_version: "22"
              run_tests: true
        
          docker:
            needs: build-test
        
            uses: ./.github/workflows/reusable-docker.yml
            with:
              image_name: github-actions-capstone
              tag: latest
            secrets:
              docker_username: ${{ secrets.DOCKER_USERNAME }}
              docker_token: ${{ secrets.DOCKER_TOKEN }}
        
          docker-sha:
            needs: build-test
        
            uses: ./.github/workflows/reusable-docker.yml
            with:
              image_name: github-actions-capstone
              tag: sha-${{ github.sha }}
            secrets:
              docker_username: ${{ secrets.DOCKER_USERNAME }}
              docker_token: ${{ secrets.DOCKER_TOKEN }}
        
          deploy:
            needs:
              - docker
              - docker-sha
        
            runs-on: ubuntu-latest
        
            environment:
              name: production
        
            steps:
              - name: Deploy
                run: |
                  echo "Deploying image: ${{ needs.docker.outputs.image_url }} to production"


Task 6: Scheduled Health Check

Create .github/workflows/health-check.yml:

1) Trigger: schedule with cron '0 */12 * * *' (every 12 hours) + workflow_dispatch for manual testing

        on:
          schedule:
            - cron: '0 */12 * * *' # Every 12 hours
          workflow_dispatch:

2) Steps:

* Pull your latest Docker image

        - name: Pull latest Docker image
                run: |
                  docker pull ${{ secrets.DOCKER_USERNAME }}/github-actions-capstone:latest{ secrets.DOCKER_TOKEN }}

* Run the container in detached mode

        - name: Run container
                run: |
                  docker run -d -app-name  -p 3000:3000 \
                    ${{ secrets.DOCKER_USERNAME }}/github-actions-capstone:latest

* Wait 5 seconds, then curl the health endpoint

        - name: Wait for application
                run: sleep 5

* Print pass/fail based on the response

        - name: Health Check
                id: health
                run: |
                  if curl -f http://localhost:3000/health; then
                    echo "status=PASSED" >> $GITHUB_OUTPUT
                  else
                    echo "status=FAILED" >> $GITHUB_OUTPUT
                    exit 1
                  fi

* Stop and remove the container

        - name: Stop and remove container
                if: always()
                run: |
                  docker stop app
                  docker rm app

3) Add a step that creates a summary using $GITHUB_STEP_SUMMARY:

        echo "## Health Check Report" >> $GITHUB_STEP_SUMMARY
        echo "- Image: myapp:latest" >> $GITHUB_STEP_SUMMARY
        echo "- Status: PASSED" >> $GITHUB_STEP_SUMMARY
        echo "- Time: $(date)" >> $GITHUB_STEP_SUMMARY

        - name: Create Health Check Summary
                if: always()
                run: |
                  echo "## Health Check Report" >> $GITHUB_STEP_SUMMARY
                  echo "- Image: ${{ secrets.DOCKER_USERNAME }}/github-actions-capstone:latest" >> $GITHUB_STEP_SUMMARY
                  echo "- Status: ${{ steps.health.outputs.status || 'FAILED' }}" >> $GITHUB_STEP_SUMMARY
                  echo "- Time: $(date)" >> $GITHUB_STEP_SUMMARY

`health-check.yml`

        name: Scheduled Health Check
        
        on:
          schedule:
            - cron: '0 */12 * * *' # Every 12 hours
          workflow_dispatch:
        
        jobs:
          health-check:
            runs-on: ubuntu-latest
        
            steps:
              - name: Log in to Docker Hub
                uses: docker/login-action@v3
                with:
                  username: ${{ secrets.DOCKER_USERNAME }}
                  password: ${{ secrets.DOCKER_TOKEN }}
        
              - name: Pull latest Docker image
                run: |
                  docker pull ${{ secrets.DOCKER_USERNAME }}/github-actions-capstone:latest
        
              - name: Run container
                run: |
                  docker run -d --name app -p 3000:3000 \
                    ${{ secrets.DOCKER_USERNAME }}/github-actions-capstone:latest
        
              - name: Wait for application
                run: sleep 5
        
              - name: Health Check
                id: health
                run: |
                  if curl -f http://localhost:3000/health; then
                    echo "status=PASSED" >> $GITHUB_OUTPUT
                  else
                    echo "status=FAILED" >> $GITHUB_OUTPUT
                    exit 1
                  fi
        
              - name: Stop and remove container
                if: always()
                run: |
                  docker stop app
                  docker rm app
        
              - name: Create Health Check Summary
                if: always()
                run: |
                  echo "## Health Check Report" >> $GITHUB_STEP_SUMMARY
                  echo "- Image: ${{ secrets.DOCKER_USERNAME }}/github-actions-capstone:latest" >> $GITHUB_STEP_SUMMARY
                  echo "- Status: ${{ steps.health.outputs.status || 'FAILED' }}" >> $GITHUB_STEP_SUMMARY
                  echo "- Time: $(date)" >> $GITHUB_STEP_SUMMARY


Task 7: Add Badges & Documentation

1) Add status badges for all your workflows to the repo README.md

2) Add a pipeline architecture diagram in your notes — draw (or describe) the flow:

        PR opened → build & test → PR checks pass
        Merge to main → build & test → Docker build & push → deploy
        Every 12 hours → health check

                    Pull Request
                         │
                         ▼
                PR Pipeline Trigger
                         │
                         ▼
               Reusable Build & Test
                         │
                         ▼
                PR Checks Successful
                         │
                         ▼
                 Merge into main
                         │
                         ▼
               Main Pipeline Trigger
                         │
                         ▼
               Reusable Build & Test
                         │
                         ▼
           Reusable Docker Build & Push
            ├── latest
            └── sha-<commit>
                         │
                         ▼
              Production Deployment
                         │
                         ▼
                 Application Running

- - -

        Every 12 Hours (Cron)
                  │
                  ▼
             Health Check Workflow
                  │
                  ▼
         Pull Latest Docker Image
                  │
                  ▼
         Start Container
                  │
                  ▼
           Curl /health Endpoint
                  │
                  ▼
           PASS / FAIL Summary
                  │
                  ▼
         Stop & Remove Container

3) Fill in your notes: What would you add next? (Slack notifications? Multi-environment? Rollback?)

Possible future improvements:

1) Slack Notifications

- Notify the team when builds fail or deployments succeed.

2) Multi-Environment Deployments

- Separate development, staging, and production environments.
- Deploy to staging automatically.
- Require approval before deploying to production.

3) Automatic Rollback

- Roll back to the previous Docker image if the health check fails after deployment.

4) Security Scanning

- Scan dependencies with Dependabot or npm audit.
- Scan Docker images using Trivy.

5) Code Quality Checks

- ESLint
- Prettier
- Unit test coverage reports

6) Container Registry Support

- Publish images to GitHub Container Registry (GHCR) in addition to Docker Hub.

7) Versioned Releases

- Automatically create Git tags and GitHub Releases for production deployments.

8) Monitoring

- Integrate Prometheus and Grafana.
- Add uptime monitoring and alerting.

# In the above I don't know have of the things right now but i'll learn and improve this small project
