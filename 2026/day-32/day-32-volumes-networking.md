Task 1: The Problem

1) Run a Postgres or MySQL container

        docker run -d -e MYSQL_ROOT_PASSWORD=root mysql:latest

2) Create some data inside it (a table, a few rows — anything)

        docker exec -it 86c bash

then we are in bash

    mysql -u root -p
    Enter password: root

then we are in mysql 

    show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | mysql              |
    | performance_schema |
    | sys                |
    +--------------------+
    4 rows in set (0.028 sec)
    
    create database imp_data;
    Query OK, 1 row affected (0.011 sec)
    
    show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | imp_data           |
    | information_schema |
    | mysql              |
    | performance_schema |
    | sys                |
    +--------------------+
    5 rows in set (0.001 sec)

3) Stop and remove the container
        
        docker stop 86c && docker rm 86c

4) Run a new one — is your data still there?

        NO

Q. Write what happened and why.

        we created a mysql-container and made a database in the sql
        then we stopped and removed the container 
        and the database which was created was also deleted from the sql 
        and we lost the data and it can't be restored


Task 2: Named Volumes

1) Create a named volume
        
        docker volume create mysql-data

        docker volume ls
        DRIVER    VOLUME NAME
        local     3e2f724006192ce49705935af0185ddd5a0730e7f4b7210ede73cd21ed6576ae
        local     6b8894fdb429ecb116a5fc59aeaffe7a19f23de2ffffb8009382f846d4813111
        local     98de639588f4a87fc1f120f64682d9c83612aa6dafe95778f65ce3a9c1cac23b
        local     a1f7ffb32f73bd64035fd0efe293737fda81a75dd5ee4884371936c99e6a9f94
        local     a9036ae7bc0b973212b04cdaab19535e9c18d02f83ae596501dd13bb6b09f5d8
        local     c680db2a578025d03bd6f51908da532cca8de0022ea02650a577d310df7019e9
        local     cf45dcdc3070fb6170fa3836a8e2a078cefc1bb76d74de12e24e363e00ee6ed6
        local     f2c11ea9f9eb105c2c3934cf00727bb4c4373d9e6a02d1a0d41aad5e94fdd56f
        local     mysql-data

        docker volume inspect mysql-data
        [
            {
                "CreatedAt": "2026-05-20T20:27:15Z",
                "Driver": "local",
                "Labels": null,
                "Mountpoint": "/var/lib/docker/volumes/mysql-data/_data",
                "Name": "mysql-data",
                "Options": null,
                "Scope": "local"
            }
        ]
        
2) Run the same database container, but this time attach the volume to it

        docker run -d -v mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSSWORD=root mysql:latest

3) Add some data, stop and remove the container

        docker exec -it 697 bash
        mysql -u root -p
        password: root
        show databases;
        +--------------------+
        | Database           |
        +--------------------+
        | information_schema |
        | mysql              |
        | performance_schema |
        | sys                |
        +--------------------+
        4 rows in set (0.037 sec)
        
        create database imp_data;
        Query OK, 1 row affected (0.031 sec)
        
        show databases;
        +--------------------+
        | Database           |
        +--------------------+
        | imp_data           |
        | information_schema |
        | mysql              |
        | performance_schema |
        | sys                |
        +--------------------+
        5 rows in set (0.016 sec)
        
        docker stop 697 && docker rm 697

4) Run a brand new container with the same volume
        
        docker run -d -v mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root mysql:latest

5) Is the data still there?

        docker exec -it cf6 bash
        
        show databases;
        +--------------------+
        | Database           |
        +--------------------+
        | imp_data           |
        | information_schema |
        | mysql              |
        | performance_schema |
        | sys                |
        +--------------------+
        5 rows in set (0.016 sec)

Verify: docker volume ls, docker volume inspect


Task 3: Bind Mounts

1) Create a folder on your host machine with an index.html file
        
        mkdir project
        vim index.html

2) Run an Nginx container and bind mount your folder to the Nginx web directory
        
        docker run -d -v /home/ubuntu/project:/usr/share/nginx/html -p 80:80 nginx:alpine

3) Access the page in your browser

output on the web page 

        Welcome to My Docker Website
        This website is running inside an Nginx container.

4) Edit the index.html on your host — refresh the browser
        
        added a heading and refresed the site and the updated html was seen on the web browser

Write in your notes: What is the difference between a named volume and a bind mount?

Bind Mount:

        - Maps a specific folder/file from the host machine into the container
        - Changes on the host appear instantly inside the container
        - Commonly used for development
        - Depends on host filesystem paths

Named Volume:

        - Managed entirely by Docker
        - Stored in Docker’s internal storage area
        - Better for persistent application data like databases
        - More portable and isolated from the host filesystem

Example:

Bind Mount:
        
        -v $(pwd):/app

Named Volume:

        -v mydata:/app/data


Task 4: Docker Networking Basics

1) List all Docker networks on your machine
        
        docker network ls

2) Inspect the default bridge network
        
        docker network inspect bridge

3) Run two containers on the default bridge — can they ping each other by name?
4) Run two containers on the default bridge — can they ping each other by IP?
Default bridge network:

        - Containers CAN communicate using IP addresses
        - Containers CANNOT usually resolve each other by container name

Reason:

        The default bridge network does not include automatic DNS service discovery.
        Container name resolution works properly on user-defined bridge networks.


Task 5: Custom Networks

1) Create a custom bridge network called my-app-net
        
        docker network create my-app-net

2) Run two containers on my-app-net

        docker run -d --name c1 --network my-app-net alpine
        docker run -d --name c2 --network my-app-net alpine

3) Can they ping each other by name now?

        yes, because the network is same for the both 
        Because user-defined bridge networks include built-in DNS resolution.
        
        docker exec c1 ping c2

4) Write in your notes: Why does custom networking allow name-based communication but the default bridge doesn't?
        
        the custom networking allows name-based communication because it has in-built DNS resolution
        AND
        the default bridge doesn't allows cause id does not include automatic DNS service discovery.


Task 6: Put It Together

1) Create a custom network
        
        docker network create prac-net

2) Run a database container (MySQL/Postgres) on that network with a volume for data

        docker run -d -v mysql-data:/var/lib/mysql --name mysql-db --network prac-net -e MYSQL_ROOT_PASSWORD=root mysql:latest

3) Run an app container (use any image) on the same network

        docker run -dit --name app-container --network prac-net alpine sh

4) Verify the app container can reach the database by container name

        docker exec app-container apk add mysql-client
        docker exec app-container ping mysql-db

        docker exec -it app-container mysql -h mysql-db -u root -p
        Enter password: root

        show databases;
        information_schema
        myapp
        mysql
        performance_schema
        sys
