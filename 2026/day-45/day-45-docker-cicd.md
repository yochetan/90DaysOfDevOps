Task 1: Prepare

1) Use the app you Dockerized on Day 36 (or any simple Dockerfile)

`Dockerfile`

        FROM node:22-alpine
        
        WORDIR /app
        
        COPY package*.json .
        
        RUN npm install
        
        COPY . .
        
        EXPOSE 3000
        
        CMD ["node", "index.js"]

2) Add the Dockerfile to your github-actions-practice repo (or create a minimal one)

3) Make sure DOCKER_USERNAME and DOCKER_TOKEN secrets are set from Day 44
