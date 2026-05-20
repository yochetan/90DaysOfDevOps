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
Create a folder on your host machine with an index.html file
Run an Nginx container and bind mount your folder to the Nginx web directory
Access the page in your browser
Edit the index.html on your host — refresh the browser
Write in your notes: What is the difference between a named volume and a bind mount?
