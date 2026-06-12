* Container commands — run, ps, stop, rm, exec, logs

      docker ps 
      docker ps -a
      docker start 'container_id'
      docker stop 'container_id'
      docker rm 'container_id'
      docker run -d -p 8080:8080 hello-app:v1 jdk-hello-app
      docker rmi 'image_name'
      docker logs 'container_id'
      docker logs -f 'container_id'
      docker exec -it webserver bash {for bash}
      docker exec -it webserver sh {for shell when bash isn't available}

* Image commands — build, pull, push, tag, ls, rm

      docker build -t jdk-hello-app .
      docker tag app:v1 user/app:v1
      docker pull nginx
      docker push user/app:v1
      docker images
      docker image ls
      docker rmi app:v1
      docker image prune
      docker image prune -a

* Volume commands — create, ls, inspect, rm

      docker volume create my-volume
      docker volume ls
      docker volume inspect my-volume
      docker volume rm my-volume
      docker volume prune

* Network commands — create, ls, inspect, connect

      docker network create my-network
      docker network ls
      docker network inspect my-network
      docker network connect my-network app
      docker network disconnect my-network app
      docker network rm my-network

* Compose commands — up, down, ps, logs, build
      
      docker compose up 
      docker compose up -d
      docker compose up --build
      docker compose down
      docker compose down -v
      docker compose ps
      docker compose ps -a
      docker compose logs
      docker compose logs -f
      docker compose build
      docker compose build --no-cache

* Cleanup commands — prune, system df
      
      docker system df
      docker system df -v
      docker image prune
      docker image prune -a
      docker container prune
      docker volume prune
      docker network prune
      docker builder prune
      docker system prune
      docker system prune -a
      docker system prune -a --volumes

* Dockerfile instructions — FROM, RUN, COPY, WORKDIR, EXPOSE, CMD, ENTRYPOINT

      FROM node:20
      
      WORKDIR /app
      
      COPY package*.json ./
      
      RUN npm install
      
      COPY . .
      
      EXPOSE 8080
      
      CMD ["npm","start"]
      
      ENTRYPOINT ["echo"]

FROM: 
      
      - Specifies the base image.

RUN:

      - Executes commands during image build.

COPY:

      - Copies files from the host into the image.

WORKDIR:

      - Sets the working directory inside the image.

EXPOSE:

      - Documents which port the application listens on.

CMD: 

      - Provides the default command when a container starts.

ENTRYPOINT:

      - Defines the executable that always runs.
