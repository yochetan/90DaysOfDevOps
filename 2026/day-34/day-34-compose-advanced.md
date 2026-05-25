Task 1: Build Your Own App Stack

Create a docker-compose.yml for a 3-service stack:

* A web app (use Python Flask, Node.js, or any language you know)
* A database (Postgres or MySQL)
* A cache (Redis)

Write a simple Dockerfile for the web app. The app doesn't need to be complex — even a "Hello World" that connects to the database is enough.

**File Path**

    mkdir multi-service-app
    cd multi-service-app
    mkdir app
    cd app
    vim .env

**.env file**

    MYSQL_ROOT_PASSWORD=rootpassword
    MYSQL_DATABASE=myapp
    MYSQL_USER=myuser
    MYSQL_PASSWORD=mypassword

**
