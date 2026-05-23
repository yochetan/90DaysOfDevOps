Task 1: Install & Verify

1) Check if Docker Compose is available on your machine
        
        sudo snap install docker

2) Verify the version
        
        docker-compose --version


Task 2: Your First Compose File

1) Create a folder `compose-basics`
        
        mkdir compose-basics

2) Write a `docker-compose.yml` that runs a single Nginx container with port mapping
        
        vim docker-compose.yml
        
        services:
                nginx:
                image: nginx:latest
                ports:
                        - "80:80"

3) Start it with `docker compose up`
        
        docker compose up

4) Access it in your browser
        
        yeah it ran properly and it is visible

5) Stop it with `docker compose down`
        
        docker compose down


Task 3: Two-Container Setup

Write a docker-compose.yml that runs:

* A WordPress container
* A MySQL container

They should:

* Be on the same network (Compose does this automatically)
* MySQL should have a named volume for data persistence
* WordPress should connect to MySQL using the service name

Start it, access WordPress in your browser, and set it up.

`docker-compose.yml` file

        services:
                db:
                        image: mysql:8.0
                        container_name: mysql-db
                        environment: 
                                MYSQL_ROOT_PASSWORD: test@123
                                MYSQL_DATABASE: wordpress
                                MYSQL_USER: wpuser
                                MYSQL_PASSWORD: wppassword
                        volumes:
                                - mysql_data:/var/lib/mysql
        
                wordpress:
                        image: wordpress:latest
                        container_name: wordpress-app
                        ports:
                                - "80:80"
                        environment:
                                WORDPRESS_DB_HOST: db:3306
                                WORDPRESS_DB_NAME: wordpress
                                WORDPRESS_DB_USER: wpuser
                                WORDPRESS_DB_PASSWORD: wppassword
                        depends_on:
                                - db

        volumes:
                mysql_data:

***Verify***: Stop and restart with `docker compose down` and `docker compose up` — is your WordPress data still there?
        
        yes the data is still there as we have a persitent volume (mysql_data)

Task 4: Compose Commands

Practice and document these:

1) Start services in detached mode
        
        docker compose up -d

2) View running services

        docker compose ps
        docker ps 
        docker ps -a
   
3) View logs of all services
        
        docker compose logs
        docker compose logs -f
        f: follows/tail/live logs

4) View logs of a specific service

        docker compose logs wordpress
        docker compose logs -f wordpress

5) Stop services without removing
        
        docker compose stop

6) Remove everything (containers, networks)
        
        docker compose down

Remove everything including volumes

        docker compose down -v 
        
7) Rebuild images if you make a change

        docker compose up --build
        docker compose up -d --build


Task 5: Environment Variables

1) Add environment variables directly in your docker-compose.yml

        services:
          db:
            image: mysql:8.0
            environment:
              MYSQL_ROOT_PASSWORD: rootpassword
              MYSQL_DATABASE: wordpress

2) Create a .env file and reference variables from it in your compose file

        vim .env

        MYSQL_ROOT_PASSWORD=rootpassword
        MYSQL_DATABASE=wordpress
        MYSQL_USER=wpuser
        MYSQL_PASSWORD=wppassword
        WORDPRESS_DB_PORT=3306

3) Verify the variables are being picked up

        services:
          db:
            image: mysql:8.0
            container_name: mysql-db
            restart: always
            environment:
              MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
              MYSQL_DATABASE: ${MYSQL_DATABASE}
              MYSQL_USER: ${MYSQL_USER}
              MYSQL_PASSWORD: ${MYSQL_PASSWORD}
            volumes:
              - mysql_data:/var/lib/mysql
        
          wordpress:
            image: wordpress:latest
            container_name: wordpress-app
            restart: always
            ports:
              - "8080:80"
            environment:
              WORDPRESS_DB_HOST: db:${WORDPRESS_DB_PORT}
              WORDPRESS_DB_NAME: ${MYSQL_DATABASE}
              WORDPRESS_DB_USER: ${MYSQL_USER}
              WORDPRESS_DB_PASSWORD: ${MYSQL_PASSWORD}
            depends_on:
              - db
        
        volumes:
          mysql_data:
