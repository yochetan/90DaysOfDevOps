Task 1: Build Your Own App Stack

Create a docker-compose.yml for a 3-service stack:

* A web app (use Python Flask, Node.js, or any language you know)
* A database (Postgres or MySQL)
* A cache (Redis)

Write a simple Dockerfile for the web app. The app doesn't need to be complex — even a "Hello World" that connects to the database is enough.

**File Path**

       multi-service-app/
    ├── app/
    │   ├── app.py
    │   ├── requirements.txt
    │   └── Dockerfile
    ├── docker-compose.yml
    └── .env
    
**.env file**

    MYSQL_ROOT_PASSWORD=rootpassword
    MYSQL_DATABASE=myapp
    MYSQL_USER=myuser
    MYSQL_PASSWORD=mypassword

**app.py**

    from flask import Flask
    import mysql.connector
    import redis
    import os
    
    app = Flask(__name__)
    
    @app.route("/")
    def home():
        db_status = "Not Connected"
        redis_status = "Not Connected"
    
        try:
            conn = mysql.connector.connect(
                host="db",
                user=os.getenv("MYSQL_USER"),
                password=os.getenv("MYSQL_PASSWORD"),
                database=os.getenv("MYSQL_DATABASE")
            )
    
            if conn.is_connected():
                db_status = "MySQL Connected"
    
            conn.close()
    
        except Exception as e:
            db_status = f"MySQL Error: {e}"
    
        try:
            r = redis.Redis(host="redis", port=6379)
            r.ping()
            redis_status = "Redis Connected"
    
        except Exception as e:
            redis_status = f"Redis Error: {e}"
    
        return f"""
        <h1>Hello from Flask!</h1>
        <p>{db_status}</p>
        <p>{redis_status}</p>
        """
    
    if __name__ == "__main__":
        app.run(host="0.0.0.0", port=5000)

**requirements.txt**

    flask
    mysql-connector-python
    redis

**Dockerfile**

    FROM python:3.11-slim
    
    WORKDIR /app
    
    COPY requirements.txt .
    
    RUN pip install --no-cache-dir -r requirements.txt
    
    COPY . .
    
    EXPOSE 5000
    
    CMD ["python", "app.py"]

**docker-compose.yml**

    services:
      web:
        build: ./app
        container_name: flask-app
        ports:
          - "5000:5000"
        environment:
          MYSQL_DATABASE: ${MYSQL_DATABASE}
          MYSQL_USER: ${MYSQL_USER}
          MYSQL_PASSWORD: ${MYSQL_PASSWORD}
        depends_on:
          - db
          - redis
    
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
    
      redis:
        image: redis:latest
        container_name: redis-cache
        restart: always
    
    volumes:
      mysql_data:

then we will build it using **docker compose**
    
    docker compose up -d --build

check the containers 
    
    docker compose ps


Task 2: depends_on & Healthchecks

1) Add depends_on to your compose file so the app starts after the database
        
        yes i used it

        web:
        build: ./app
        container_name: flask-app
        ports:
          - "5000:5000"
        environment:
          MYSQL_DATABASE: ${MYSQL_DATABASE}
          MYSQL_USER: ${MYSQL_USER}
          MYSQL_PASSWORD: ${MYSQL_PASSWORD}
        depends_on:
          - db
          - redis

2) Add a healthcheck on the database service
    
        yes that would be good
         
        healthcheck:
              test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
              interval: 10s
              timeout: 5s
              retries: 5
              start_period: 30s # Give MySQL time to initialize on first boot

3) Use depends_on with condition: service_healthy so the app waits for the database to be truly ready, not just started

        depends_on:
              db:
                condition: service_healthy

Test: Bring everything down and up — does the app wait for the DB?
    
    yes it does as we have used healtcheck for database and the app doesn't works if the database don't gets started

Task 3: Restart Policies

1) Add restart: always to your database service

        db:
                image: mysql:8.0
                container_name: mysql-db
                restart: always

2) Manually kill the database container — does it come back?

        yes it comes back as we specified a persitance volume

        db:
                image: mysql:8.0
                container_name: mysql-db
                restart: always
                volumes:
                  - mysql_data:/var/lib/mysql
            
3) Try restart: on-failure — how is it different?
        
        restart only if the container exits with an error
        does NOT restart after a clean/normal exit

4) Write in your notes: When would you use each restart policy?

a) restart: no (default)

When to use: 

    temporary/testing containers
    one-time scripts
    debugging containers
    containers you want to stop permanently after exit

Example:

    Running a script once and exiting

b) restart: always

When to use: 
    
    production web servers
    databases
    APIs
    Redis
    long-running services
    applications that must stay online

Behavior: 
    
    always restart after crash
    restart after reboot
    restart even after normal exit

Example:

    Nginx, MySQL, Flask API

c) restart: on-failure

When to use:
    
    workers
    retryable jobs
    scripts that may temporarily fail
    migration tasks

Behavior:

    restart only when exit code is non-zero
    do not restart after successful completion

Example:

    Background worker that retries failed jobs

d) restart: unless-stopped

When to use:

    same as always, but you want manual stops respected

Behavior:

    restart automatically after crashes/reboots
    but if you manually stop it, Docker keeps it stopped

Example:

    Personal servers or development environments


Task 4: Custom Dockerfiles in Compose

1) Instead of using a pre-built image for your app, use build: in your compose file to build from a Dockerfile

        yeah i did in the first task
        web:
        build: ./app
        container_name: flask-app

2) Make a code change in your app

**app.py**

    from flask import Flask
    import mysql.connector
    import redis
    import os
    
    app = Flask(__name__)
    
    @app.route("/")
    def home():
        db_status = "Not Connected"
        redis_status = "Not Connected"
    
        try:
            conn = mysql.connector.connect(
                host="db",
                user=os.getenv("MYSQL_USER"),
                password=os.getenv("MYSQL_PASSWORD"),
                database=os.getenv("MYSQL_DATABASE")
            )
    
            if conn.is_connected():
                db_status = "MySQL Connected"
    
            conn.close()
    
        except Exception as e:
            db_status = f"MySQL Error: {e}"
    
        try:
            r = redis.Redis(host="redis", port=6379)
            r.ping()
            redis_status = "Redis Connected"
    
        except Exception as e:
            redis_status = f"Redis Error: {e}"
    
        return f"""
        <h1>Hello from Flask!</h1>
        <p>{db_status}</p>
        <p>{redis_status}</p>
        """
    
    if __name__ == "__main__":
        app.run(host="0.0.0.0", port=5000)

3) Rebuild and restart with one command

        docker compose up --build

Task 5: Named Networks & Volumes

1) Define explicit networks in your compose file instead of relying on the default
        
        docker network create flask-app-network

2) Define named volumes for database data
        
        docker volume create flask-app-db-volume

3) Add labels to your services for better organization

        Docker labels are key-value metadata attached to containers, images, volumes, or services.

They help with:

    organization
    filtering
    monitoring
    automation
    reverse proxies (like Traefik)
    documentation

Example:

    services:
      web:
        build: ./app
        container_name: flask-app
        ports:
          - "5000:5000"
        labels:
          com.project.name: "multi-service-app"
          com.project.service: "web"
          com.project.environment: "development"

Task 6: Scaling (Bonus)

1) Try scaling your web app to 3 replicas using docker compose up --scale

Try scaling your web service:

    docker compose up -d --scale web=3

Then check containers:

    docker compose ps

You’ll likely see:

    one container running normally
    other replicas failing or not starting

Example:

    Error starting userland proxy:
    Bind for 0.0.0.0:5000 failed:
    port is already allocated

2) What happens? What breaks?

Your web service probably has:
    
    ports:
      - "5000:5000"

This means:
    
    Host machine port 5000
            ↓
    Container port 5000

Problem:
    
    Only ONE container can use host port 5000.

When scaling to 3 replicas:

    web-1 → tries host port 5000
    web-2 → tries host port 5000
    web-3 → tries host port 5000

- But the host machine cannot bind the same port multiple times.

- So only the first container succeeds.

3) Write in your notes: Why doesn't simple scaling work with port mapping?

- A host port can only be attached to one container at a time.

- Port mappings expose containers to the outside world through the host machine.

Example:

    5000:5000

means:

    HOST_PORT:CONTAINER_PORT

Since:
    
    the host has only one port 5000
    multiple containers cannot share it

    simple horizontal scaling fails.

Why simple scaling with ports fails:
    
    - Each container tries to bind the same host port
    - A host port can only belong to one process/container
    - Scaling works internally on Docker networks
    - External access requires a load balancer or reverse proxy

Real systems solve this using:

    - Nginx
    - Traefik
    - HAProxy
    - Kubernetes Services
