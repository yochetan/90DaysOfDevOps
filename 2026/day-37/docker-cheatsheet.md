* Container commands — run, ps, stop, rm, exec, logs

      docker ps 
      docker ps -a


docker start 'id'
docker stop 'id'
docker rm 'id'
docker build -t jdk-hello-app .
docker run -d -p 8080:8080 hello-app:v1 jdk-hello-app
docker rmi 'image_name'
