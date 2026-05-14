Task 1: Your First Dockerfile

1) Create a folder called my-first-image

        mkdir my-first-image

2) Inside it, create a Dockerfile that:

        touch Dockerfile

* Uses ubuntu as the base image
* Installs curl
* Sets a default command to print "Hello from my custom image!"
      
      FROM ubuntu
      
      RUN apt update && apt install -y curl
      
      CMD ["echo", "Hello from my custom image"]

3) Build the image and tag it my-ubuntu:v1
        
        docker -t build my-ubuntu:v1

4) Run a container from your image
        
        docker run my-ubuntu:v1
   
Verify: The message prints on docker run

output: 

      Hello from my custom image


Task 2: Dockerfile Instructions

Create a new Dockerfile that uses all of these instructions:

- FROM — base image
- RUN — execute commands during build
- COPY — copy files from host to image
- WORKDIR — set working directory
- EXPOSE — document the port
- CMD — default command
        
        FROM ubuntu
        RUN apt update && apt install -y bash
        COPY app.sh .
        WORKDIR /app
        EXPOSE 8080
        CMD ["bash", "app.sh"]

Build and run it. Understand what each line does.
        
        docker build -t my-demo-app:v1 .
        docker run my -demo-app:v1


Task 3: CMD vs ENTRYPOINT

1) Create an image with CMD ["echo", "hello"] — run it, then run it with a custom command. What happens?
        
        FROM ubuntu
        CMD ["echo", "hello"]

2) Create an image with ENTRYPOINT ["echo"] — run it, then run it with additional arguments. What happens?
        
        FROM ubuntu
        ENTRYPOINT ["echo"]

3) Write in your notes: When would you use CMD vs ENTRYPOINT?

CMD vs ENTRYPOINT
        
        +--------------------------+--------------------------------+
        | CMD                      | ENTRYPOINT                     |
        +--------------------------+--------------------------------+
        | Default command          | Fixed main command             |
        | Can be overridden easily | Harder to override             |
        | Good for defaults        | Good for executable containers |
        +--------------------------+--------------------------------+


Task 4: Build a Simple Web App Image

1) Create a small static HTML file (index.html) with any content

        <!DOCTYPE html>
        <html>
        <head>
            <title>My Website</title>
            <style>
                body {
                    font-family: Arial, sans-serif;
                    background-color: #f4f4f4;
                    text-align: center;
                    margin-top: 100px;
                }
                h1 {
                    color: #333;
                }
            </style>
        </head>
        <body>
            <h1>Welcome to My Docker Website 🚀</h1>
            <p>This website is running inside an Nginx container.</p>
        </body>
        </html>

2) Write a Dockerfile that:
- Uses nginx:alpine as base
- Copies your index.html to the Nginx web directory
        
        FROM nginx:alpine
        COPY index.html /usr/share/nginx/html/index.html

3) Build and tag it my-website:v1
        
        docker build -t my-website:v1

4) Run it with port mapping and access it in your browser
        
        docker run -d -p 80:80 --name mywebsite my-website:v1


Task 5: .dockerignore

1) Create a .dockerignore file in one of your project folders


2) Add entries for: node_modules, .git, *.md, .env


3) Build the image — verify that ignored files are not included


Task 6: Build Optimization

1) Build an image, then change one line and rebuild — notice how Docker uses cache
2) Reorder your Dockerfile so that frequently changing lines come last
3) Write in your notes: Why does layer order matter for build speed?
