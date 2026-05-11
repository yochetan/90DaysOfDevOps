Task 1: What is Docker?
    
    Docker is a platform used to build, package, and run applications inside containers.
    It helps developers run applications consistently on any system without “it works on my machine” problems.

Research and write short notes on:

What is a container and why do we need them?
    
    A container is a lightweight package that includes:
    
    Application code
    Libraries
    Dependencies
    Runtime
    Configuration files
    
    Everything needed to run the app is bundled together.

    Why do we need containers?
    Problems before containers
    
    Applications often failed because:
    
    Different OS environments
    Missing dependencies
    Different software versions
    Configuration mismatches
    
    Example:
    
    Works on developer laptop ❌
    Fails on server

Containers vs Virtual Machines — what's the real difference?
    
    Containers
    Share the host OS kernel
    Lightweight
    Start very fast
    Use less RAM and CPU
    Best for microservices and DevOps
    
    Virtual Machines (VMs)
    Include a full guest OS
    Heavyweight
    Slower startup
    Use more resources
    Better isolation

What is the Docker architecture? (daemon, client, images, containers, registry)

    Docker follows a client-server architecture.
    
    Main components:
    
    Docker Client
    Docker Daemon
    Docker Images
    Docker Containers
    Docker Registry

Draw or describe the Docker architecture in your own words.

    +-------------------+
    |   Docker Client   |
    | docker commands   |
    +---------+---------+
              |
              v
    +-------------------+
    |   Docker Daemon   |
    |  (dockerd)        |
    +----+--------+-----+
         |        |
         |        |
         v        v
    +---------+  +------------+
    | Images  |  | Containers |
    +---------+  +------------+
         |
         v
    +-------------------+
    | Docker Registry   |
    | (Docker Hub)      |
    +-------------------+

    Simple Explanation of Architecture
    1. User runs a Docker command
    2. Docker Client sends request to Docker Daemon
    3. Daemon pulls image from Docker Registry if needed
    4. Daemon creates and runs the container
    5. Container runs the application


Task 2: Install Docker

1) Install Docker on your machine (or use a cloud instance)

        sudo apt-get update
        sudo apt-get install docker.io

2) Verify the installation
        
        docker --version
        Docker version 29.1.3, build 29.1.3-0ubuntu4.1
        sudo usermod -aG docker ubuntu
        newgrp docker
        docker ps            

3) Run the hello-world container

        docker run hello-world

4) Read the output carefully — it explains what just happened

        Hello from Docker!
        This message shows that your installation appears to be working correctly.
        
        To generate this message, Docker took the following steps:
         1. The Docker client contacted the Docker daemon.
         2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
            (amd64)
         3. The Docker daemon created a new container from that image which runs the
            executable that produces the output you are currently reading.
         4. The Docker daemon streamed that output to the Docker client, which sent it
            to your terminal.


Task 3: Run Real Containers

1) Run an Nginx container and access it in your browser
        
        docker run -d -p 80:80 nginx

2) Run an Ubuntu container in interactive mode — explore it like a mini Linux machine
        
        docker run -itd ubuntu
        docker exec -it d3b0853a4fb8 bash
        
        then we will be in bash shell
        i ran these commands init
        
        apt-get update
        apt-get install nginx
        service nginx status (it showed nginx wasn't running)
        service nginx start (it started to run)

3) List all running containers  
        
        docker ps

4) List all containers (including stopped ones)
        
        docker ps -a

5) Stop and remove a container
        
        docker stop 1a5de677068d && docker rm 1a5de677068d


Task 4: Explore

1) Run a container in detached mode — what's different?
        
        docker run -d -e MYSQL_ROOT_PASSWORD=test@123 mysql
        
        detached mode means running the image in the background

2) Give a container a custom name

        docker run -d --name mysql-demo -e MYSQL_ROOT_PASSWORD=test@123 mysql

3) Map a port from the container to your host

        docker run -d -p 80:80 nginx

4) Check logs of a running container

        docker logs 633e0505b876

5) Run a command inside a running container

        docker exec -it ubuntu bash
