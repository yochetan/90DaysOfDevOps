Task 1: Understand the Problem

1) Create a Deployment with 3 replicas using nginx

`nginx-deployment.yml`

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
      labels:
        app: nginx
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - name: nginx
            image: nginx:1.14.2
            ports:
            - containerPort: 80
    

2) Check the pod names — they are random (app-xyz-abc)

3) Delete a pod and notice the replacement gets a different random name

This is fine for web servers but not for databases where you need stable identity.

| Feature          | Deployment         | StatefulSet                            |
|------------------|--------------------|----------------------------------------|
| Pod names        | Random             | Stable, ordered (app-0, app-1)         |
| Startup order    | All at once        | Ordered: pod-0, then pod-1, then pod-2 |
| Storage          | Shared PVC         | Each pod gets its own PVC              |
| Network identity | No stable hostname | Stable DNS per pod                     |

Delete the Deployment before moving on.

Verify: Why would random pod names be a problem for a database cluster?

