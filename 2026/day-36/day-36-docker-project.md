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

- we will got to this path /express/examples/hello-world/Dockerfile

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

- then we will build it
      
      docker build -t express-hello:v1 .

- then we will run it
      
      docker run -d -p 3000:3000 --name express-app express-hello:v1

2) Use a multi-stage build if applicable

3) Use a non-root user

4) Keep the image small — use alpine or slim base images

5) Add a .dockerignore file

Build and test it locally.
