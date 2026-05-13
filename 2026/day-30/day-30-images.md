Task 1: Docker Images

1) Pull the nginx, ubuntu, and alpine images from Docker Hub
        
        docker pull nginx
        
        docker pull ubuntu
        
        docker pull alpine

2) List all images on your machine — note the sizes

        docker images
        
        IMAGE                ID             DISK USAGE   CONTENT SIZE   EXTRA
        alpine:latest        5b10f432ef3d       13.1MB         3.95MB
        hello-world:latest   f9078146db2e       25.9kB         9.49kB
        nginx:latest         e1318da3b3c0        240MB         65.9MB
        ubuntu:latest        f3d28607ddd7        160MB         45.3MB    U


3) Compare ubuntu vs alpine — why is one much smaller?

        ubuntu size is 160MB and alpine is 13.1MB
        +----------------------+--------------------------+
        | Ubuntu               | Alpine                   |
        +----------------------+--------------------------+
        | Full-featured OS     | Minimal OS               |
        | More utilities       | Fewer utilities          |
        | Larger libraries     | Lightweight libraries    |
        | Better compatibility | Optimized for small size |
        +----------------------+--------------------------+

4) Inspect an image — what information can you see?

        docker image inspect nginx

      Metadata
   
        Image ID
        Creation date
        OS and architecture
        Environment variables

     Configuration

        Default command (CMD)
        Entrypoint
        Exposed ports
        Working directory

     Storage Details

        Layers
        Size
        Digest/hash

     Example Fields

        "Architecture": "amd64"
        "Os": "linux"
        "Cmd": ["nginx", "-g", "daemon off;"]
        "ExposedPorts": {
            "80/tcp": {}
        }

5) Remove an image you no longer need

        docker rmi nginx


Task 2: Image Layers

1) Run docker image history nginx — what do you see?

        docker image history nginx
        IMAGE          CREATED      CREATED BY                                      SIZE      COMMENT
        1881968aff6f   5 days ago   CMD ["nginx" "-g" "daemon off;"]                0B        buildkit.dockerfile.v0
        <missing>      5 days ago   STOPSIGNAL SIGQUIT                              0B        buildkit.dockerfile.v0
        <missing>      5 days ago   EXPOSE map[80/tcp:{}]                           0B        buildkit.dockerfile.v0
        <missing>      5 days ago   ENTRYPOINT ["/docker-entrypoint.sh"]            0B        buildkit.dockerfile.v0
        <missing>      5 days ago   COPY 30-tune-worker-processes.sh /docker-ent…   16.4kB    buildkit.dockerfile.v0
        <missing>      5 days ago   COPY 20-envsubst-on-templates.sh /docker-ent…   12.3kB    buildkit.dockerfile.v0
        <missing>      5 days ago   COPY 15-local-resolvers.envsh /docker-entryp…   12.3kB    buildkit.dockerfile.v0
        <missing>      5 days ago   COPY 10-listen-on-ipv6-by-default.sh /docker…   12.3kB    buildkit.dockerfile.v0
        <missing>      5 days ago   COPY docker-entrypoint.sh / # buildkit          8.19kB    buildkit.dockerfile.v0
        <missing>      5 days ago   RUN /bin/sh -c set -x     && groupadd --syst…   86.7MB    buildkit.dockerfile.v0
        <missing>      5 days ago   ENV DYNPKG_RELEASE=1~trixie                     0B        buildkit.dockerfile.v0
        <missing>      5 days ago   ENV PKG_RELEASE=1~trixie                        0B        buildkit.dockerfile.v0
        <missing>      5 days ago   ENV ACME_VERSION=0.3.1                          0B        buildkit.dockerfile.v0
        <missing>      5 days ago   ENV NJS_RELEASE=1~trixie                        0B        buildkit.dockerfile.v0
        <missing>      5 days ago   ENV NJS_VERSION=0.9.6                           0B        buildkit.dockerfile.v0
        <missing>      5 days ago   ENV NGINX_VERSION=1.29.8                        0B        buildkit.dockerfile.v0
        <missing>      5 days ago   LABEL maintainer=NGINX Docker Maintainers <d…   0B        buildkit.dockerfile.v0
        <missing>      8 days ago   # debian.sh --arch 'amd64' out/ 'trixie' '@1…   87.4MB    debuerreotype 0.17


2) Each line is a layer. Note how some layers show sizes and some show 0B

        11 layers has 0B and 5 layers are in kB and remaining 2 layers are in MB

3) Write in your notes: What are layers and why does Docker use them?

In Docker, layers are read-only filesystem changes stacked on top of each other to build an image.

Each Docker image is made of multiple layers:

    Base OS Layer
    + Package Layer
    + App Dependencies Layer
    + Application Code Layer

Docker uses layers to make images faster, smaller, reusable, and efficient.


Task 3: Container Lifecycle

Practice the full lifecycle on one container:

1) Create a container (without starting it)

        docker create --name mycontainer nginx

2) Start the container
        
        docker start mycontainer

3) Pause it and check status
        
        docker pause mycontainer

4) Unpause it
        
        docker unpause mycontainer

5) Stop it
        
        docker stop mycontainer

6) Restart it
        
        docker restart mycontainer

7) Kill it
        
        docker kill mycontainer

8) Remove it
        
        docker rm mycontainer

Check docker ps -a after each step — observe the state changes.
        
        docker ps -a


Task 4: Working with Running Containers

1) Run an Nginx container in detached mode
        
        docker run -d --name nginx-container -p 80:80 nginx

2) View its logs
        
        docker logs nginx-container

3) View real-time logs (follow mode)

        docker logs -f nginx-container

4) Exec into the container and look around the filesystem
        
        docker exec -it nginx-container bash

5) Run a single command inside the container without entering it

        docker exec nginx-container ls /usr/share/nginx/html

6) Inspect the container — find its IP address, port mappings, and mounts

        docker inspect nginx-container


Task 5: Cleanup

1) Stop all running containers in one command
        
        docker stop $(docker ps -q)  

2) Remove all stopped containers in one command
        
        docker container prune  

3) Remove unused images
        
        docker image prune  

4) Check how much disk space Docker is using
        
        docker system df
