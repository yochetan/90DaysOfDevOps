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
Run a new one — is your data still there?
Write what happened and why.
