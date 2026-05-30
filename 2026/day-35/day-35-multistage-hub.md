Task 1: The Problem with Large Images

1) Write a simple Go, Java, or Node.js app (even a "Hello World" is fine)

**app.js**

        const http = require("http");
        
        const server = http.createServer((req, res) => {
          res.writeHead(200, { "Content-Type": "text/plain" });
          res.end("Hello from Node.js Docker App!\n");
        });
        
        server.listen(3000, () => {
          console.log("Server running on port 3000");
        });

**package.json**

        {
          "name": "node-docker-app",
          "version": "1.0.0",
          "main": "app.js"
        }

2) Create a Dockerfile that builds and runs it in a single stage

        FROM node:20
        
        WORKDIR /app
        
        COPY . .
        
        EXPOSE 3000
        
        CMD ["node", "app.js"]

3) Build the image and check its size

        docker build -t node-hello-app .

* Note down the size — you'll compare it later

        docker images
   
        IMAGE                   ID             DISK USAGE   CONTENT SIZE   EXTRA
        node-hello-app:latest   ea225aa07d9c        400MB         100MB        
        node:22                 968df39aedce        400MB         100.6MB


Task 2: Multi-Stage Build

1) Rewrite the Dockerfile using multi-stage build:

* Stage 1: Build the app (install dependencies, compile)
        
        #Stage 1 - Build
        FROM node:22-alpine AS Builder
        
        WORKDIR /app
        
        COPY package*.json
        
        RUN npm install --production
        
        COPY . .

* Stage 2: Copy only the built artifact into a minimal base image (alpine, distroless, or scratch)
        
        #Stage 2 - Runtime 
        
        FROM node:22-alpine
        
        WORKDIR /app
        
        COPY --from=Builder /app .
        
        EXPOSE 3000
        
        CMD ["node", "app.js"]

2) Build the image and check its size again
        
        docker build -f Dockerfile.multi -t node-hello-app-mini .

3) Compare the two sizes

        docker images
   
        IMAGE                        ID             DISK USAGE   CONTENT SIZE   EXTRA
        node-hello-app-mini:latest   bbad7d6dafd7        230MB         57.5MB       

Write in your notes: Why is the multi-stage image so much smaller?

Why is the multi-stage image smaller?

        1. Build tools are excluded from the final image.
        2. Source code can be excluded if only compiled artifacts are needed.
        3. Package managers (npm, Maven, Gradle, etc.) remain in the build stage.
        4. Temporary files and caches are not copied into the runtime image.
        5. The final image contains only what is needed to run the application.
        
        Benefits:
        - Smaller image size
        - Faster downloads and deployments
        - Reduced attack surface
        - Less disk usage
        - Better security

Task 3: Push to Docker Hub

1) Create a free account on Docker Hub (if you don't have one)
        
        DONE!!!

2) Log in from your terminal



3) Tag your image properly: yourusername/image-name:tag

4) Push it to Docker Hub

5) Pull it on a different machine (or after removing locally) to verify

Task 4: Docker Hub Repository

1) Go to Docker Hub and check your pushed image

2) Add a description to the repository

3) Explore the tags tab — understand how versioning works

4) Pull a specific tag vs latest — what happens?


Task 5: Image Best Practices
Apply these to one of your images and rebuild:

1) Use a minimal base image (alpine vs ubuntu — compare sizes)

2) Don't run as root — add a non-root USER in your Dockerfile

3) Combine RUN commands to reduce layers

4) Use specific tags for base images (not latest)

Check the size before and after.
