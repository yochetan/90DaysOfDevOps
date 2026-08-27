Task 1: Deploy the Application

1) First, create a Deployment that you will expose with Services. Create app-deployment.yaml:

        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: web-app
          labels:
            app: web-app
        spec:
          replicas: 3
          selector:
            matchLabels:
              app: web-app
          template:
            metadata:
              labels:
                app: web-app
            spec:
              containers:
              - name: nginx
                image: nginx:1.25
                ports:
                - containerPort: 80

kubectl apply -f app-deployment.yaml
kubectl get pods -o wide
Note the individual Pod IPs. These will change if pods restart — that is the problem Services fix.

Verify: Are all 3 pods running? Note down their IP addresses.

