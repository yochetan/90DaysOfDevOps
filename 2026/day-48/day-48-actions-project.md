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

2) Inputs: python_version (or node_version), run_tests (boolean, default: true)

3) Steps:

* Check out code

* Set up the language runtime

* Install dependencies

* Run tests (only if run_tests is true)

* Set output: test_result with value passed or failed

This workflow does NOT deploy — it only builds and tests.
