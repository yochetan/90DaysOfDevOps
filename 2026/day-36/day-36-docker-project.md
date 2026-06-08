Task 1: Pick Your App

Choose one of these (or use your own project):

* A Python Flask/Django app with a database
* A Node.js Express app with MongoDB
* A static website served by Nginx with a backend API
* Any app from your GitHub that doesn't have Docker yet

If you don't have an app, clone a simple open-source one and Dockerize it.

    I took this one `https://github.com/expressjs/express.git`

Task 2: Write the Dockerfile

1) Create a Dockerfile for your application

- we will go to this path /express/examples/hello-world/Dockerfile

`Dockerfile`
    
    FROM node:22-alpine
    
    WORDIR /app
    
    COPY package*.json .
    
    RUN npm install
    
    COPY . .
    
    EXPOSE 3000
    
    CMD ["node", "index.js"]

- first install npm packages and express
    
      npm init -y
      
      npm install express

- updated the index.js

        var express = require('../../');
        TO
        var express = require('express');

- then we will build it
      
      docker build -t express-hello:v1 .

- then we will run it
      
      docker run -d -p 3000:3000 --name express-app express-hello:v1

2) Use a multi-stage build if applicable

`Dockerfile.multistage`

    #Stage 1: build
    FROM node:22-alpine AS build
    
    WORKDIR /app
    
    COPY package*.json .
    
    RUN npm install
    
    COPY . .
    
    #Stage 2: Runtime
    FROM node:22-alpine 
    
    WORKDIR /app
    
    COPY --from=build /app .
    
    EXPOSE 3000
    
    CMD ["node", "index.js"]

3) Use a non-root user

        FROM node:22-alpine
        
        WORKDIR /app
        
        COPY package*.json .
        
        RUN npm install
        
        COPY . .
        
        USER node
        
        EXPOSE 3000
        
        CMD ["node", "index.js"]

4) Keep the image small — use alpine or slim base images
        
        node:22-alpine

5) Add a .dockerignore file

`.dockerignore`

    node_modules
    .git
    .gitignore
    Dockerfile
    README.md

Build and test it locally.

and builded and tested

I can see `Hello World` when tested


Task 3: Add Docker Compose

Write a `docker-compose.yml` that includes:

1) Your app service (built from Dockerfile)
        
        services:
          app:
            build: .
            container_name: express-app
            restart: unless-stopped
            ports:
              - "3000:3000"
            environment:
              DB_HOST: ${DB_HOST}
              DB_PORT: ${DB_PORT}
              DB_NAME: ${DB_NAME}
            depends_on:
              mongodb:
                condition: service_healthy
            networks:
              - app-network
        
          mongodb:
            image: mongo:8.0
            container_name: mongodb
            restart: unless-stopped
            environment:
              MONGO_INITDB_ROOT_USERNAME: ${MONGO_INITDB_ROOT_USERNAME}
              MONGO_INITDB_ROOT_PASSWORD: ${MONGO_INITDB_ROOT_PASSWORD}
            volumes:
              - mongo_data:/data/db
            healthcheck:
              test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
              interval: 10s
              timeout: 5s
              retries: 5
              start_period: 20s
            networks:
              - app-network
        
        volumes:
          mongo_data:
        
        networks:
          app-network:
            driver: bridge

2) A database service (Postgres, MySQL, MongoDB — whatever your app needs)
        
         mongodb:
            image: mongo:8.0
            container_name: mongodb
            restart: unless-stopped
            environment:
              MONGO_INITDB_ROOT_USERNAME: ${MONGO_INITDB_ROOT_USERNAME}
              MONGO_INITDB_ROOT_PASSWORD: ${MONGO_INITDB_ROOT_PASSWORD}

3) Volumes for database persistence
        
        volumes:
          mongo_data:

4) A custom network

        networks:
          app-network:
            driver: bridge

5) Environment variables for configuration (use .env file)

        MONGO_INITDB_ROOT_USERNAME=admin
        MONGO_INITDB_ROOT_PASSWORD=password1234
        
        DB_HOST=mongodb
        DB_PORT=27017
        DB_NAME=myapp

6) Healthchecks on the database

        healthcheck:
              test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
              interval: 10s
              timeout: 5s
              retries: 5
              start_period: 20s
      
Run docker compose up and verify everything works together.
    
    docker compose up


Task 4: Ship It

1) Tag your app image
        
        docker image tag hello-world-app:latest stupidgenuis/hello-world-app:latest

2) Push it to Docker Hub
        
        docker push stupidgenuis/hello-world-app:latest

3) Share the Docker Hub link

        https://hub.docker.com/repository/docker/stupidgenuis/hello-world-app/general

4) Write a README.md in your project with:

* What the app does
        
        displays `Hello World`

* How to run it with Docker Compose
        
        docker compose up -d 

* Any environment variables needed
        
        environments of mongodb (not needed but i just took it for an example)

Task 5: Test the Whole Flow

1) Remove all local images and containers
        
        yes did 

2) Pull from Docker Hub and run using only your compose file
        
        yes pulled and it works

3) Does it work fresh? If not — fix it until it does
        
        yesss

* What app you chose and why

        Basic Express structure
        Easy to containerize
        Good for learning Node.js Docker workflows

Your Dockerfile (with comments explaining each line)

FROM node:22-alpine

    pulls node image and used alpine as it small and the work is done 

WORKDIR /app
    
    creates a /app directory where all the files will go 

COPY package*.json .
    
    copies all the json packages in the /app folder

RUN npm install
    
    install nodes

COPY . .
    
    again copies everything in the folder

USER node

    switches to node user so incase if a breach or hacker hacks the container he has limited permissions and the damage is less and it is more secured

EXPOSE 3000
    
    exposing port 3000 (letting the developer know which port the app will runs

CMD ["node", "index.js"]
    
    executing index.js using node

Challenges you faced and how you solved them

Final image size
    
    241 MB

Docker Hub link

    https://hub.docker.com/repository/docker/stupidgenuis/hello-world-app/general
