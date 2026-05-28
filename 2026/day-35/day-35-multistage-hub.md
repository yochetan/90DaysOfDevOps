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

4) Note down the size — you'll compare it later

