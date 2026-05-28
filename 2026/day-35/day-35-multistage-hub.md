Task 1: The Problem with Large Images

1) Write a simple Go, Java, or Node.js app (even a "Hello World" is fine)



2) Create a Dockerfile that builds and runs it in a single stage

        FROM node:20
        
        WORKDIR /app
        
        COPY . .
        
        EXPOSE 3000
        
        CMD ["node", "app.js"]

3) Build the image and check its size

docker 

4) Note down the size — you'll compare it later
