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

        docker login

3) Tag your image properly: yourusername/image-name:tag
        
        docker image tag ai-bankapp:latest stupidgenuis/ai-bankapp:latest

4) Push it to Docker Hub

        docker push stupidgenuis/ai-bankapp:latest

5) Pull it on a different machine (or after removing locally) to verify
        
        did it locally and it works 

Task 4: Docker Hub Repository

1) Go to Docker Hub and check your pushed image
        
        yeah it is there

2) Add a description to the repository
        
        yeah wrote: image for ai-bankapp

3) Explore the tags tab — understand how versioning works
        
        latest
        v1
        v1.1
        v2
        dev

4) Pull a specific tag vs latest — what happens?

Docker Tags and Versioning
        
        - A tag is a label attached to an image.
        - Multiple tags can point to different image versions.
        - 'latest' is not automatically the newest image; it is simply a tag.

        - Pulling a specific tag always retrieves that exact version.
        - Pulling 'latest' retrieves whatever image is currently tagged as latest.
        - Production environments should use versioned tags (v1.0, v1.1, v2.0) rather than latest.
        - Tags make it easy to roll back to previous versions.

Task 5: Image Best Practices
Apply these to one of your images and rebuild:

1) Use a minimal base image (alpine vs ubuntu — compare sizes)
        
        Image	                Approx Size
        Ubuntu/Jammy JDK	400–700 MB
        Alpine JRE	        100–200 MB

2) Don't run as root — add a non-root USER in your Dockerfile

Instead of:

        root

create a dedicated user:

        RUN addgroup -S spring && \
            adduser -S spring -G spring
        
        USER spring

Verify:

        docker exec -it container-name whoami

Expected:

        spring

Benefits:

        Better security
        Limits damage if app is compromised
        Follows container best practices

3) Combine RUN commands to reduce layers

Less efficient:

        RUN sed -i 's/\r$//' mvnw
        
        RUN chmod +x mvnw
        
        RUN ./mvnw clean package -DskipTests

* Each RUN creates a layer.

Better:

        RUN sed -i 's/\r$//' mvnw && \
            chmod +x mvnw && \
            ./mvnw clean package -DskipTests

Benefits:

        Fewer layers
        Smaller image
        Faster builds

4) Use specific tags for base images (not latest)

Avoid:

        FROM maven:latest

Use:

        FROM maven:3.9.9-eclipse-temurin-21-alpine

and

        FROM eclipse-temurin:21-jre-alpine

Benefits:

        Reproducible builds
        No surprise upgrades
        Easier debugging

Check the size before and after.

Build:

        docker build -t bankapp-optimized .

Check:

        docker images

Example:
        
        REPOSITORY          SIZE
        bankapp-old         720MB
        bankapp-optimized   230MB
