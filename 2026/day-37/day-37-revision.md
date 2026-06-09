Self-Assessment Checklist

Mark yourself honestly — can do, shaky, or haven't done:

 Run a container from Docker Hub (interactive + detached)

    # Interactive container
    docker run -it ubuntu:latest bash

    # Detached container
    docker run -d nginx
 
 List, stop, remove containers and images

    list: docker ps / docker ps -a
    stop: docker stop 'id'
    remove containers: docker rm 'id'
    remove images: docker rmi 'id'
 
 Explain image layers and how caching works

 
 
 Write a Dockerfile from scratch with FROM, RUN, COPY, WORKDIR, CMD

    FROM node:22-alpine

    WORKDIR /app

    COPY packages*.json .

    RUN npm install

    COPY . .

    CMD ["node","index.js"]
 
 Explain CMD vs ENTRYPOINT

 *CMD

    WE USE THIS WHEN THE COMMANDS ARE FIXED AND WILL NOT BE CHANGED IN THE FUTURE 

 *ENTRYPOINT

    IN THIS WE KNOW WE ARE GONNA ADD MORE COMMANDS SO WE CAN APPEND THOSE COMMANDS
 
 Build and tag a custom image

    docker build -t express-hello-app .
 
 Create and use named volumes

    docker create volume mongo-volume

    docker run -d -v mongo-volume express-hello-app
 
 Use bind mounts

 
 Create custom networks and connect containers

 
 Write a docker-compose.yml for a multi-container app


 
 Use environment variables and .env files in Compose


 
 Write a multi-stage Dockerfile

 
 Push an image to Docker Hub


 
 Use healthchecks and depends_on
