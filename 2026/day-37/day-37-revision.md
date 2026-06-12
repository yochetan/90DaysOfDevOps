## Self-Assessment Checklist

Mark yourself honestly — can do, shaky, or haven't done:

* Run a container from Docker Hub (interactive + detached)

      # Interactive container
      docker run -it ubuntu:latest bash
  
      # Detached container
      docker run -d nginx
 
* List, stop, remove containers and images

      list: docker ps / docker ps -a
      stop: docker stop 'id'
      remove containers: docker rm 'id'
      remove images: docker rmi 'id'
 
* Explain image layers and how caching works

      Images Layers: 
      A Docker image is made up of multiple read-only layers stacked on top of each other.
      Each instruction in a Dockerfile (such as RUN, COPY, or ADD) usually creates a new layer.

      Example:
      
      FROM ubuntu:22.04
      RUN apt-get update
      RUN apt-get install -y nginx
      COPY index.html /var/www/html/
      CMD ["nginx", "-g", "daemon off;"]
      
      Layers created:
      
      Base Ubuntu layer (FROM)
      Package index update (RUN apt-get update)
      Nginx installation (RUN apt-get install)
      Copy website files (COPY)
      Metadata layer (CMD)
      When a container starts, Docker adds a writable container layer on top of these read-only image layers.

      Docker Layer Caching:
      Docker stores previously built layers in its cache.
      
      During a rebuild, Docker checks each instruction:
      
      - If nothing has changed, Docker reuses the cached layer.
      - If something changed, Docker rebuilds that layer and all layers after it.
 
* Write a Dockerfile from scratch with FROM, RUN, COPY, WORKDIR, CMD

      FROM node:22-alpine
  
      WORKDIR /app
  
      COPY packages*.json .
  
      RUN npm install
  
      COPY . .
  
      CMD ["node","index.js"]
   
* Explain CMD vs ENTRYPOINT

 - CMD

          WE USE THIS WHEN THE COMMANDS ARE FIXED AND WILL NOT BE CHANGED IN THE FUTURE
         
            Purpose	            Provides a default command	
            Can be overridden?	Completely overridden by command-line arguments	
            Typical use             Default behavior	

 - ENTRYPOINT

          IN THIS WE KNOW WE ARE GONNA ADD MORE COMMANDS SO WE CAN APPEND THOSE COMMANDS
      
            Purpose	            Defines the main executable
            Can be overridden?	Not overridden; arguments are appended
            Typical use		      Fixed container behavior
 
* Build and tag a custom image

          docker build -t express-hello-app .
 
* Create and use named volumes

          docker volume create my-volume
      
          docker run -d --name nginx-container -v my-volume:/usr/share/nginx/html nginx
 
* Use bind mounts

      A bind mount lets a container access files and directories from your host machine directly.
      Changes made on the host are immediately visible inside the container, and vice versa.
      
      Syntax:
      docker run --mount type=bind,source=<host-path>,target=<container-path> <image>
      
      | Bind Mount vs Volume    |                 |                   |
      |-------------------------|-----------------|-------------------|
      | Feature                 | Bind Mount      | Docker Volume     |
      | Stored where?           | Host filesystem | Managed by Docker |
      | Easy to edit from host? | Yes             | Not directly      |
      | Portable?               | Less portable   | More portable     |
      | Performance             | Good            | Good              |
      | Common use              | Development     | Production data   |

* Create custom networks and connect containers

      docker network create n1
      
      docker run -d --network n1 --name hello-app hello-image
 
* Write a docker-compose.yml for a multi-container app

       services:
         app:
           build: .
           container_name: express-app
           restart: unless-stopped
           ports:
             - "3000:3000"
           environment:
             DB_HOST: ${DB_HOST}
             DB_PORT: ${DB_PORT}
             DB_NAME: ${DB_NAME}
           depends_on:
             mongodb:
               condition: service_healthy
           networks:
             - app-network
       
         mongodb:
           image: mongo:8.0
           container_name: mongodb
           restart: unless-stopped
           environment:
             MONGO_INITDB_ROOT_USERNAME: ${MONGO_INITDB_ROOT_USERNAME}
             MONGO_INITDB_ROOT_PASSWORD: ${MONGO_INITDB_ROOT_PASSWORD}
           volumes:
             - mongo_data:/data/db
           healthcheck:
             test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
             interval: 10s
             timeout: 5s
             retries: 5
             start_period: 20s
           networks:
             - app-network
       
       volumes:
         mongo_data:
       
       networks:
         app-network:
           driver: bridge
  
* Use environment variables and .env files in Compose

environment variables in Compose

       environment:
             MONGO_INITDB_ROOT_USERNAME: ${MONGO_INITDB_ROOT_USERNAME}
             MONGO_INITDB_ROOT_PASSWORD: ${MONGO_INITDB_ROOT_PASSWORD}

`.env`

       MONGO_INITDB_ROOT_USERNAME=admin
       MONGO_INITDB_ROOT_PASSWORD=password1234
       
       DB_HOST=mongodb
       DB_PORT=27017
       DB_NAME=myapp
 
* Write a multi-stage Dockerfile
      
      #Stage 1: build
      FROM node:22-alpine AS build
      
      WORKDIR /app
      
      COPY package*.json .
      
      RUN npm install
      
      COPY . .
      
      #Stage 2: Runtime
      FROM node:22-alpine 
      
      WORKDIR /app
      
      COPY --from=build /app .
      
      EXPOSE 3000
      
      CMD ["node", "index.js"]
 
* Push an image to Docker Hub

       docker push stupidgenuis/hello-world-app:latest
 
* Use healthchecks and depends_on

      healthcheck:
             test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
             interval: 10s
             timeout: 5s
             retries: 5
             start_period: 20s
      
      depends_on:
             mongodb:
