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

--

Task 2: Dockerfile Instructions

Create a new Dockerfile that uses all of these instructions:

- FROM — base image
- RUN — execute commands during build
- COPY — copy files from host to image
- WORKDIR — set working directory
- EXPOSE — document the port
- CMD — default command


Build and run it. Understand what each line does.
