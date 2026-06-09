Self-Assessment Checklist

Mark yourself honestly — can do, shaky, or haven't done:

 Run a container from Docker Hub (interactive + detached)

    docker run -i -d 
 
 List, stop, remove containers and images

    list: docker ps / docker ps -a
    stop: docker stop 'id'
    remove containers: docker rm 'id'
    remove images: docker rmi 'id'
 
 Explain image layers and how caching works

 
 
 Write a Dockerfile from scratch with FROM, RUN, COPY, WORKDIR, CMD

 FROM 
 
 Explain CMD vs ENTRYPOINT
 Build and tag a custom image
 Create and use named volumes
 Use bind mounts
 Create custom networks and connect containers
 Write a docker-compose.yml for a multi-container app
 Use environment variables and .env files in Compose
 Write a multi-stage Dockerfile
 Push an image to Docker Hub
 Use healthchecks and depends_on
