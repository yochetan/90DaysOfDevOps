Task 1: Prepare

1) Use the app you Dockerized on Day 36 (or any simple Dockerfile)

`app.py`

        print("Hello from Docker!")

2) Add the Dockerfile to your github-actions-practice repo (or create a minimal one)

`Dockerfile`

        FROM python:3.12-slim

        WORKDIR /app
        
        COPY . .
        
        CMD ["python", "app.py"]

3) Make sure DOCKER_USERNAME and DOCKER_TOKEN secrets are set from Day 44

        yeah they are set

Task 2: Build the Docker Image in CI

Create .github/workflows/docker-publish.yml that:

1) Triggers on push to main

        on:
                  push:
                    branches:
                      - main

2) Checks out the code

        - name: Checkout Repository
                        uses: actions/checkout@v4

3) Builds the Docker image and tags it

        - name: Build Docker Image
                        run: |
                          docker build -t myapp:${{ github.sha }} .

`docker-publish.yml`

        name: Docker Build
        
        on:
          push:
            branches:
              - main
        
        jobs:
          docker-build:
            runs-on: ubuntu-latest
        
            steps:
              - name: Checkout Repository
                uses: actions/checkout@v4
        
              - name: Build Docker Image
                run: |
                  docker build -t myapp:${{ github.sha }} .

Verify: Check the build step logs — does the image build successfully?

        yes all the steps are completed successfully


Task 3: Push to Docker Hub

Add steps to:

1) Log in to Docker Hub using your secrets

        - name: Log in to Docker Hub
                        uses: docker/login-action@v3
                        with:
                          username: ${{ secrets.DOCKERHUB_USERNAME }}
                          password: ${{ secrets.DOCKERHUB_TOKEN }}

2) Tag the image as username/repo:latest and also username/repo:sha-<short-commit-hash>

        - name: Build Docker Image
                        run: |
                          docker build \
                            -t ${{ secrets.DOCKERHUB_USERNAME }}/github-actions-practice:latest \
                            -t ${{ secrets.DOCKERHUB_USERNAME }}/github-actions-practice:sha-${{ env.SHORT_SHA }} \

3) Push both tags

              - name: Push latest tag
                run: docker push ${{ secrets.DOCKERHUB_USERNAME }}/github-actions-practice:latest
        
              - name: Push SHA tag
                run: docker push ${{ secrets.DOCKERHUB_USERNAME }}/github-actions-practice:sha-${{ env.SHORT_SHA }}

`docker-publish.yml`

        name: Docker Publish
        
        on:
          push:
            branches:
              - main
        
        jobs:
          docker-publish:
            runs-on: ubuntu-latest
        
            steps:
              - name: Checkout Repository
                uses: actions/checkout@v4
        
              - name: Generate Short SHA
                run: echo "SHORT_SHA=${GITHUB_SHA::7}" >> $GITHUB_ENV
        
              - name: Log in to Docker Hub
                uses: docker/login-action@v3
                with:
                  username: ${{ secrets.DOCKERHUB_USERNAME }}
                  password: ${{ secrets.DOCKERHUB_TOKEN }}
        
              - name: Build Docker Image
                run: |
                  docker build \
                    -t ${{ secrets.DOCKERHUB_USERNAME }}/github-actions-practice:latest \
                    -t ${{ secrets.DOCKERHUB_USERNAME }}/github-actions-practice:sha-${{ env.SHORT_SHA }} \
                    .
        
              - name: Push latest tag
                run: docker push ${{ secrets.DOCKERHUB_USERNAME }}/github-actions-practice:latest
        
              - name: Push SHA tag
                run: docker push ${{ secrets.DOCKERHUB_USERNAME }}/github-actions-practice:sha-${{ env.SHORT_SHA }}

Verify: Go to Docker Hub — is your image there with both tags?
        
        yeah, i can see both the tags 


Task 4: Only Push on Main

Add a condition so the push step only runs on the main branch — not on feature branches or PRs.

        - name: Push latest image
          if: github.ref == 'refs/heads/main'
          run: docker push ${{ secrets.DOCKERHUB_USERNAME }}/github-actions-practice:latest
        
        - name: Push SHA image
          if: github.ref == 'refs/heads/main'
          run: docker push ${{ secrets.DOCKERHUB_USERNAME }}/github-actions-practice:sha-${{ steps.vars.outputs.short_sha }}

Test it: push to a feature branch and verify the image is built but NOT pushed.

         yeah did, it was built but not pushed to docker hub


Task 5: Add a Status Badge

1) Get the badge URL for your docker-publish workflow from the Actions tab

        [![Docker Publish](https://github.com/yochetan/github-actions-practice/actions/workflows/docker-publish.yml/badge.svg)](https://github.com/yochetan/github-actions-practice/actions/workflows/docker-publish.yml)

2) Add it to your README.md

`README.md`

        [![Docker Publish](https://github.com/yochetan/github-actions-practice/actions/workflows/docker-publish.yml/badge.svg)](https://github.com/yochetan/github-actions-practice/actions/workflows/docker-publish.yml)

3) Push — the badge should show green

        yes, it is showing green and passing


Task 6: Pull and Run It

1) On your local machine (or a cloud server), pull the image you just pushed

        docker pull stupidgenuis/github-actions-practice:latest

2) Run it

        docker run --rm stupidgenuis/github-actions-practice:latest

Output:

        Hello from Docker!

3) Confirm it works

- The container starts without errors.
- Your application behaves as expected.
- docker ps shows the container running (for long-running apps).

Write in your notes: What is the full journey from git push to a running container?

- I make changes to my application and commit them to Git.
- I run git push to push the code to GitHub.
- GitHub Actions detects the push and starts the docker-publish.yml workflow.
- The workflow checks out the repository on a GitHub-hosted runner.
- Docker Hub login is performed using repository secrets.
- Docker builds the image from the Dockerfile.
- The image is tagged (for example, latest and sha-<short-commit-hash>).
- On the main branch, the workflow pushes the tagged image to Docker Hub.
- On another machine or server, I run docker pull to download the image from Docker Hub.
- I start the application using docker run.
- Docker creates and starts a container from the image, and the application is now running.

CI/CD flow summary

        Code Change
            │
            ▼
        git add → git commit → git push
            │
            ▼
        GitHub Repository
            │
            ▼
        GitHub Actions Workflow
            │
            ├── Checkout Code
            ├── Build Docker Image
            ├── Tag Image
            └── Push to Docker Hub (main branch only)
            │
            ▼
        Docker Hub
            │
            ▼
        docker pull
            │
            ▼
        docker run
            │
            ▼
        Running Container
