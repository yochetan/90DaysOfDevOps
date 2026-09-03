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
        
2) Check the pod names — they are random (`app-xyz-abc`)
        
        nginx-deployment-54fd4d6d4c-9zg96   0/1     ContainerCreating   0          14s
        nginx-deployment-54fd4d6d4c-kzfh4   0/1     ContainerCreating   0          14s
        nginx-deployment-54fd4d6d4c-s5czf   0/1     ContainerCreating   0          14s

3) Delete a pod and notice the replacement gets a different random name

- kubectl delete pod nginx-deployment-54fd4d6d4c-9zg96

        pod "nginx-deployment-54fd4d6d4c-9zg96" deleted from default namespace

- kubectl get pods                                    

        NAME                                READY   STATUS    RESTARTS   AGE
        nginx-deployment-54fd4d6d4c-5cz2d   1/1     Running   0          2s
        nginx-deployment-54fd4d6d4c-kzfh4   1/1     Running   0          10m
        nginx-deployment-54fd4d6d4c-s5czf   1/1     Running   0          10m

This is fine for web servers but not for databases where you need stable identity.

| Feature          | Deployment         | StatefulSet                            |
|------------------|--------------------|----------------------------------------|
| Pod names        | Random             | Stable, ordered (app-0, app-1)         |
| Startup order    | All at once        | Ordered: pod-0, then pod-1, then pod-2 |
| Storage          | Shared PVC         | Each pod gets its own PVC              |
| Network identity | No stable hostname | Stable DNS per pod                     |

Delete the Deployment before moving on.

- kubectl delete deploy nginx-deployment              

        deployment.apps "nginx-deployment" deleted from default namespace

Verify: Why would random pod names be a problem for a database cluster?

    a normal application can usually say "give me any healthy Pod." A database cluster often needs to say "I am node 1, and this is my data and cluster identity." That's the key reason random Pod names can be problematic.

---

Task 2: Create a Headless Service

1) Write a Service manifest with `clusterIP: None` — this is a Headless Service

`headless-service.yml`

        apiVersion: v1
        kind: Service
        metadata:
          name: web-app-headless
        spec:
          clusterIP: None
          selector:
            app: web-app
          ports:
            - port: 80
              targetPort: 80

2) Set the selector to match the labels you will use on your StatefulSet pods

        selector:
                    app: web-app

3) Apply it and confirm CLUSTER-IP shows `None`

- kubectl apply -f .\headless-service.yml

        service/web-app-headless created

- kubectl get svc web-app-headless

        NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
        web-app-headless   ClusterIP   None         <none>        80/TCP    102s

A Headless Service creates individual DNS entries for each pod instead of load-balancing to one IP. StatefulSets require this.

Verify: What does the CLUSTER-IP column show?

    None

---

Task 3: Create a StatefulSet

1) Write a StatefulSet manifest with `serviceName` pointing to your Headless Service

        apiVersion: apps/v1
        kind: StatefulSet
        metadata:
          name: web
        spec:
          selector:
            matchLabels:
              app: nginx 
          serviceName: "nginx"

2) Set replicas to 3, use the nginx image

        replicas: 3 
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

3) Add a `volumeClaimTemplates` section requesting 100Mi of ReadWriteOnce storage

                volumeMounts:
                - name: www
                  mountPath: /usr/share/nginx/html
          volumeClaimTemplates:
          - metadata:
              name: www
            spec:
              accessModes:
              - ReadWriteOnce
              resources:
                requests:
                  storage: 100Mi

5) Apply and watch: `kubectl get pods -l <your-label> -w`

- kubectl apply -f .\statefulset.yml

        statefulset.apps/web created

- kubectl get pods -l app=nginx -w

        NAME    READY   STATUS    RESTARTS   AGE
        web-0   1/1     Running   0          93s
        web-1   1/1     Running   0          88s
        web-2   1/1     Running   0          82s

Observe ordered creation — `web-0` first, then `web-1` after `web-0` is Ready, then `web-2`.

Check the PVCs: `kubectl get pvc` — you should see `web-data-web-0`,`web-data-web-1`, `web-data-web-2` (names follow the pattern `<template-name>-<pod-name>)`.

- kubectl get pvc                 

        NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
        www-web-0   Bound    pvc-45414f43-8add-4ea2-a0f8-262c7c24dcbb   100Mi      RWO            standard       <unset>                 3m18s
        www-web-1   Bound    pvc-fd8341f0-c9c1-4a7d-be68-1e0804ba2b0f   100Mi      RWO            standard       <unset>                 3m13s
        www-web-2   Bound    pvc-0913c69a-950a-46e4-9231-5a2cfc027938   100Mi      RWO            standard       <unset>                 3m7s
        
Verify: What are the exact pod names and PVC names?

- pod names
        
        web-0
        web-1
        web-2

- PVC names
        
        www-web-0
        www-web-1
        www-web-2

---

Task 4: Stable Network Identity

Each StatefulSet pod gets a DNS name: `<pod-name>.<service-name>.<namespace>.svc.cluster.local`

1) Run a temporary busybox pod and use `nslookup` to resolve `web-0.<your-headless-service>.default.svc.cluster.local`

- kubectl exec dns-test -- nslookup web-0.nginx.default.svc.cluster.local

        Server:         10.96.0.10
        Address:        10.96.0.10:53
        
        
        Name:   web-0.nginx.default.svc.cluster.local
        Address: 10.244.1.5

2) Do the same for `web-1` and `web-2`

- kubectl exec dns-test -- nslookup web-1.nginx.default.svc.cluster.local

        Server:         10.96.0.10
        Address:        10.96.0.10:53
        
        
        Name:   web-1.nginx.default.svc.cluster.local
        Address: 10.244.2.5

- kubectl exec dns-test -- nslookup web-2.nginx.default.svc.cluster.local

        Server:         10.96.0.10
        Address:        10.96.0.10:53
        
        
        Name:   web-2.nginx.default.svc.cluster.local
        Address: 10.244.1.7

3) Confirm the IPs match `kubectl get pods -o wide`

- kubectl get pods -o wide

        NAME       READY   STATUS    RESTARTS   AGE     IP           NODE                     NOMINATED NODE   READINESS GATES
        dns-test   1/1     Running   0          5m38s   10.244.2.6   chetan-cluster-worker    <none>           <none>
        web-0      1/1     Running   0          13m     10.244.1.5   chetan-cluster-worker2   <none>           <none>
        web-1      1/1     Running   0          13m     10.244.2.5   chetan-cluster-worker    <none>           <none>
        web-2      1/1     Running   0          13m     10.244.1.7   chetan-cluster-worker2   <none>           <none>

Verify: Does the nslookup IP match the pod IP?

    yes.

---

Task 5: Stable Storage — Data Survives Pod Deletion

1) Write unique data to each pod: `kubectl exec web-0 -- sh -c "echo 'Data from web-0' > /usr/share/nginx/html/index.html"`

- kubectl exec web-0 -- sh -c "echo 'Data from web-0' > /usr/share/nginx/html/index.html"

- kubectl exec web-0 -- cat /usr/share/nginx/html/index.html       

        Data from web-0       

2) Delete `web-0`: `kubectl delete pod web-0`

- kubectl delete pod web-0

        pod "web-0" deleted from default namespace

3) Wait for it to come back, then check the data — it should still be "Data from web-0"

- kubectl get pods 

        NAME    READY   STATUS    RESTARTS   AGE
        web-0   1/1     Running   0          15s
        web-1   1/1     Running   0          19m
        web-2   1/1     Running   0          19m

The new pod reconnected to the same PVC.

- kubectl exec web-0 -- cat /usr/share/nginx/html/index.html

        Data from web-0

Verify: Is the data identical after pod recreation?

    yeah it is.

---

Task 6: Ordered Scaling

1) Scale up to 5: `kubectl scale statefulset web --replicas=5` — pods create in order (web-3, then web-4)

- kubectl scale statefulset web --replicas=5

        statefulset.apps/web scaled

- kubectl get pods                          

        NAME    READY   STATUS    RESTARTS   AGE
        web-0   1/1     Running   0          4m8s
        web-1   1/1     Running   0          23m
        web-2   1/1     Running   0          22m
        web-3   1/1     Running   0          8s
        web-4   0/1     Pending   0          3s

2) Scale down to 3 — pods terminate in reverse order (web-4, then web-3)

- kubectl scale statefulset web --replicas=3

        statefulset.apps/web scaled

- kubectl get pods

        NAME    READY   STATUS    RESTARTS   AGE
        web-0   1/1     Running   0          4m27s
        web-1   1/1     Running   0          23m
        web-2   1/1     Running   0          23m

3) Check `kubectl get pvc` — all five PVCs still exist. Kubernetes keeps them on scale-down so data is preserved if you scale back up.

- kubectl get pvc

        NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
        www-web-0   Bound    pvc-45414f43-8add-4ea2-a0f8-262c7c24dcbb   100Mi      RWO            standard       <unset>                 23m
        www-web-1   Bound    pvc-fd8341f0-c9c1-4a7d-be68-1e0804ba2b0f   100Mi      RWO            standard       <unset>                 23m
        www-web-2   Bound    pvc-0913c69a-950a-46e4-9231-5a2cfc027938   100Mi      RWO            standard       <unset>                 23m
        www-web-3   Bound    pvc-3d661fcb-feb4-451c-8204-f860c215430f   100Mi      RWO            standard       <unset>                 53s
        www-web-4   Bound    pvc-9e751dc5-f5aa-41a5-b424-1f50a5d70462   100Mi      RWO            standard       <unset>                 48s

Verify: After scaling down, how many PVCs exist?

    5

---

Task 7: Clean Up

1) Delete the StatefulSet and the Headless Service

- kubectl delete statefulset web  

        statefulset.apps "web" deleted from default namespace

- kubectl delete service nginx  

        service "nginx" deleted from default namespace

2) Check `kubectl get pvc` — PVCs are still there (safety feature)

- kubectl get pvc

        NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
        www-web-0   Bound    pvc-45414f43-8add-4ea2-a0f8-262c7c24dcbb   100Mi      RWO            standard       <unset>                 30m
        www-web-1   Bound    pvc-fd8341f0-c9c1-4a7d-be68-1e0804ba2b0f   100Mi      RWO            standard       <unset>                 29m
        www-web-2   Bound    pvc-0913c69a-950a-46e4-9231-5a2cfc027938   100Mi      RWO            standard       <unset>                 29m
        www-web-3   Bound    pvc-3d661fcb-feb4-451c-8204-f860c215430f   100Mi      RWO            standard       <unset>                 7m2s
        www-web-4   Bound    pvc-9e751dc5-f5aa-41a5-b424-1f50a5d70462   100Mi      RWO            standard       <unset>                 6m57s

3) Delete PVCs manually

- kubectl delete pvc www-web-0

        persistentvolumeclaim "www-web-0" deleted from default namespace

- kubectl delete pvc www-web-1

        persistentvolumeclaim "www-web-1" deleted from default namespace

- kubectl delete pvc www-web-2

        persistentvolumeclaim "www-web-2" deleted from default namespace

- kubectl delete pvc www-web-3

        persistentvolumeclaim "www-web-3" deleted from default namespace

- kubectl delete pvc www-web-4

        persistentvolumeclaim "www-web-4" deleted from default namespace

Verify: Were PVCs auto-deleted with the StatefulSet?

    No, they weren't.
