Task 1: Create the Namespace (Day 52)

1) Create a `capstone` namespace

`namespace.yml`

        apiVersion: v1
        kind: Namespace
        metadata:
          name: capstone

- kubectl apply -f .\namespace.yml              

        namespace/capstone created

2) Set it as your default: `kubectl config set-context --current --namespace=capstone`

- kubectl config set-context --current --namespace=capstone

        Context "kind-chetan-cluster" modified.

---

Task 2: Deploy MySQL (Days 54-56)

1) Create a Secret with `MYSQL_ROOT_PASSWORD`, `MYSQL_DATABASE`, `MYSQL_USER`, and `MYSQL_PASSWORD` using `stringData`

`mysql-secret.yml`

        apiVersion: v1
        kind: Secret
        metadata:
          name: mysql-secret
        type: Opaque
        stringData:
          MYSQL_ROOT_PASSWORD: rootpassword
          MYSQL_DATABASE: myapp
          MYSQL_USER: appuser
          MYSQL_PASSWORD: apppassword

- kubectl apply -f mysql-secret.yml 

        secret/mysql-secret created

2) Create a Headless Service (`clusterIP: None`) for MySQL on port 3306

`mysql-service.yml`

        apiVersion: v1
        kind: Service
        metadata:
          name: mysql
        spec:
          clusterIP: None
          selector:
            app: mysql
          ports:
            - port: 3306
              targetPort: 3306

- kubectl apply -f mysql-service.yml

        service/mysql created

3) Create a StatefulSet for MySQL with:

- Image: `mysql:8.0`

- `envFrom` referencing the Secret

- Resource requests (cpu: 250m, memory: 512Mi) and limits (cpu: 500m, memory: 1Gi)

- A `volumeClaimTemplates` section requesting 1Gi of storage, mounted at `/var/lib/mysql`

`mysql-statefulset.yml`

        apiVersion: apps/v1
        kind: StatefulSet
        metadata:
          name: mysql
        spec:
          serviceName: mysql
          replicas: 1
          selector:
            matchLabels:
              app: mysql
        
          template:
            metadata:
              labels:
                app: mysql
            spec:
              containers:
                - name: mysql
                  image: mysql:8.0
        
                  ports:
                    - containerPort: 3306
                      name: mysql
        
                  envFrom:
                    - secretRef:
                        name: mysql-secret
        
                  resources:
                    requests:
                      cpu: 250m
                      memory: 512Mi
                    limits:
                      cpu: 500m
                      memory: 1Gi
        
                  volumeMounts:
                    - name: mysql-data
                      mountPath: /var/lib/mysql
        
          volumeClaimTemplates:
            - metadata:
                name: mysql-data
              spec:
                accessModes:
                  - ReadWriteOnce
                resources:
                  requests:
                    storage: 1Gi

- kubectl apply -f .\mysql-statefulset.yml

        statefulset.apps/mysql created

4) Verify MySQL works: `kubectl exec -it mysql-0 -- mysql -u <user> -p<password> -e "SHOW DATABASES;"`

- kubectl exec -it mysql-0 -- mysql -u appuser -papppassword -e "SHOW DATABASES;"

mysql: [Warning] Using a password on the command line interface can be insecure.

        +--------------------+
        | Database           |
        +--------------------+
        | information_schema |
        | performance_schema |
        | wordpress          |
        +--------------------+

Verify: Can you see the wordpress database?

        yes

---

Task 3: Deploy WordPress (Days 52, 54, 57)

1) Create a ConfigMap with `WORDPRESS_DB_HOST` set to `mysql-0.mysql.capstone.svc.cluster.local:3306` and `WORDPRESS_DB_NAME`

`wordpress-configmap.yml`

        apiVersion: v1
        kind: ConfigMap
        metadata:
          name: wordpress-config
        data:
          WORDPRESS_DB_HOST: mysql-0.mysql.capstone.svc.cluster.local:3306
          WORDPRESS_DB_NAME: wordpress

- kubectl apply -f wordpress-configmap.yml 

        configmap/wordpress-config created

2) Create a Deployment with 2 replicas using `wordpress:latest` that:

- Uses `envFrom` for the ConfigMap

- Uses `secretKeyRef` for `WORDPRESS_DB_USER` and `WORDPRESS_DB_PASSWORD` from the MySQL Secret

- Has resource requests and limits

- Has a liveness probe and readiness probe on `/wp-login.php` port 80

`wordpress-deployment.yml`

        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: wordpress
          namespace: capstone
        spec:
          replicas: 2
          selector:
            matchLabels:
              app: wordpress
          template:
            metadata:
              labels:
                app: wordpress
            spec:
              containers:
                - name: wordpress
                  image: wordpress:latest
                  ports:
                    - containerPort: 80
        
                  envFrom:
                    - configMapRef:
                        name: wordpress-config
        
                  env:
                    - name: WORDPRESS_DB_USER
                      valueFrom:
                        secretKeyRef:
                          name: mysql-secret
                          key: MYSQL_USER
        
                    - name: WORDPRESS_DB_PASSWORD
                      valueFrom:
                        secretKeyRef:
                          name: mysql-secret
                          key: MYSQL_PASSWORD
        
                  resources:
                    requests:
                      cpu: "100m"
                      memory: "128Mi"
                    limits:
                      cpu: "500m"
                      memory: "512Mi"
        
                  livenessProbe:
                    httpGet:
                      path: /wp-login.php
                      port: 80
                    initialDelaySeconds: 30
                    periodSeconds: 10
                    timeoutSeconds: 5
                    failureThreshold: 3
        
                  readinessProbe:
                    httpGet:
                      path: /wp-login.php
                      port: 80
                    initialDelaySeconds: 20
                    periodSeconds: 10
                    timeoutSeconds: 5
                    failureThreshold: 3

- kubectl apply -f wordpress-deployment.yml 

        deployment.apps/wordpress created

3) Wait until both pods show `1/1 Running`

- kubectl get pods   

        NAME                         READY   STATUS    RESTARTS   AGE
        mysql-0                      1/1     Running   0          14m
        wordpress-5ff669bb88-5snnb   1/1     Running   0          3m51s
        wordpress-5ff669bb88-wrlkn   1/1     Running   0          3m51s

Verify: Are both WordPress pods running and ready?

        yes

---

Task 4: Expose WordPress (Day 53)

1) Create a NodePort Service on port 30080 targeting the WordPress pods

`wordpress-service.yml`
        
        apiVersion: v1
        kind: Service
        metadata:
          name: wordpress
          namespace: capstone
        spec:
          type: NodePort
          selector:
            app: wordpress
          ports:
            - port: 80
              targetPort: 80
              nodePort: 30080

- kubectl apply -f wordpress-service.yml 

        service/wordpress created

2) Access WordPress in your browser:

- Minikube: `minikube service wordpress -n capstone`

- Kind: `kubectl port-forward svc/wordpress 8080:80 -n capstone`

- kubectl port-forward svc/wordpress 8080:80 -n capstone

        Forwarding from 127.0.0.1:8080 -> 80
        Forwarding from [::1]:8080 -> 80
        Handling connection for 8080
        Handling connection for 8080
        Handling connection for 8080

3) Complete the setup wizard and create a blog post

        Yes

Verify: Can you see the WordPress setup page?

        Yes

---

Task 5: Test Self-Healing and Persistence

1) Delete a WordPress pod — watch the Deployment recreate it within seconds. Refresh the site.

- kubectl delete pod wordpress-5ff669bb88-5snnb -n capstone

        pod "wordpress-5ff669bb88-5snnb" deleted from capstone namespace

- kubectl get pods -n capstone -w                          
        
        NAME                         READY   STATUS    RESTARTS   AGE
        mysql-0                      1/1     Running   0          54m
        wordpress-5ff669bb88-nrbl6   0/1     Running   0          6s
        wordpress-5ff669bb88-wrlkn   1/1     Running   0          44m
        wordpress-5ff669bb88-nrbl6   1/1     Running   0          24s
        
2) Delete the MySQL pod: `kubectl delete pod mysql-0 -n capstone` — watch the StatefulSet recreate it

- kubectl delete pod mysql-0 -n capstone

        pod "mysql-0" deleted from capstone namespace

- kubectl get pods -n capstone -w       

        NAME                         READY   STATUS    RESTARTS   AGE
        mysql-0                      1/1     Running   0          2s
        wordpress-5ff669bb88-nrbl6   1/1     Running   0          2m11s
        wordpress-5ff669bb88-wrlkn   1/1     Running   0          46m

3) After MySQL recovers, refresh WordPress — your blog post should still be there

- WordPress is recreated by the Deployment.
- MySQL is recreated by the StatefulSet.
- MySQL's data is stored on a PersistentVolume/PersistentVolumeClaim, not inside the Pod itself.
- When mysql-0 is recreated, it reconnects to the same persistent storage, so the WordPress database—and therefore your blog post—remains.

Verify: After deleting both pods, is your blog post still there?

        Yes, the blog post is still there.

---

Task 6: Set Up HPA (Day 58)

1) Write an HPA manifest targeting the WordPress Deployment with CPU at 50%, min 2, max 10 replicas

`wordpress-hpa.yml`

        apiVersion: autoscaling/v2
        kind: HorizontalPodAutoscaler
        metadata:
          name: wordpress-hpa
          namespace: capstone
        spec:
          scaleTargetRef:
            apiVersion: apps/v1
            kind: Deployment
            name: wordpress
          minReplicas: 2
          maxReplicas: 10
          behavior:
            scaleUp:
              stabilizationWindowSeconds: 0
            scaleDown:
              stabilizationWindowSeconds: 300
          metrics:
            - type: Resource
              resource:
                name: cpu
                target:
                  type: Utilization
                  averageUtilization: 50

2) Apply and check: `kubectl get hpa -n capstone`

- kubectl apply -f .\wordpress-hpa.yml      

        horizontalpodautoscaler.autoscaling/wordpress-hpa created

- kubectl get hpa -n capstone         

        NAME            REFERENCE              TARGETS              MINPODS   MAXPODS   REPLICAS   AGE
        wordpress-hpa   Deployment/wordpress   cpu: <unknown>/50%   2         10        2          2s

3) Run `kubectl get all -n capstone` for the complete picture

- kubectl get all -n capstone

        NAME                             READY   STATUS    RESTARTS   AGE
        pod/mysql-0                      1/1     Running   0          8m17s
        pod/wordpress-5ff669bb88-nrbl6   1/1     Running   0          10m
        pod/wordpress-5ff669bb88-wrlkn   1/1     Running   0          54m
        
        NAME                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
        service/mysql       ClusterIP   None           <none>        3306/TCP       81m
        service/wordpress   NodePort    10.96.239.72   <none>        80:30080/TCP   48m
        
        NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/wordpress   2/2     2            2           54m
        
        NAME                                   DESIRED   CURRENT   READY   AGE
        replicaset.apps/wordpress-5ff669bb88   2         2         2       54m
        
        NAME                     READY   AGE
        statefulset.apps/mysql   1/1     65m
        
        NAME                                                REFERENCE              TARGETS              MINPODS   MAXPODS   REPLICAS   AGE
        horizontalpodautoscaler.autoscaling/wordpress-hpa   Deployment/wordpress   cpu: <unknown>/50%   2         10        2          35s

Verify: Does the HPA show correct min/max and target?

        Yes, it does

---

Task 7: (Bonus) Compare with Helm (Day 59)

1) Install WordPress using `helm install wp-helm bitnami/wordpress` in a separate namespace

- kubectl create namespace wp-helm

        namespace/wp-helm created

- helm install wp-helm bitnami/wordpress

        NAME: wp-helm
        LAST DEPLOYED: Mon Sep  7 03:50:15 2026
        NAMESPACE: capstone
        STATUS: deployed
        REVISION: 1
        TEST SUITE: None
        NOTES:
        CHART NAME: wordpress
        CHART VERSION: 33.0.12
        APP VERSION: 7.1.0

2) Compare: how many resources did each approach create? Which gives more control?

- kubectl get all -n wp-helm

        NAME                                     READY   STATUS     RESTARTS   AGE
        pod/wp-helm-mariadb-0                    0/1     Init:0/1   0          18s
        pod/wp-helm-wordpress-5b7bdb8cd5-j5pzs   0/1     Running    0          18s
        
        NAME                               TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
        service/wp-helm-mariadb            ClusterIP      10.96.136.62    <none>        3306/TCP                     18s
        service/wp-helm-mariadb-headless   ClusterIP      None            <none>        3306/TCP                     18s
        service/wp-helm-wordpress          LoadBalancer   10.96.214.124   <pending>     80:31848/TCP,443:31314/TCP   18s
        
        NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/wp-helm-wordpress   0/1     1            0           18s
        
        NAME                                           DESIRED   CURRENT   READY   AGE
        replicaset.apps/wp-helm-wordpress-5b7bdb8cd5   1         1         0       18s
        
        NAME                               READY   AGE
        statefulset.apps/wp-helm-mariadb   0/1     18s

- Manual Kubernetes YAML

        More direct/fine-grained control.        

- Helm

        More automation and convenience.

- Kubernetes YAML gives more low-level control, while Helm gives more automation, consistency, and easier configuration of complex applications.

- Helm is especially useful when an application has many related Kubernetes resources and dependencies.

3) Clean up the Helm deployment

- helm uninstall wp-helm -n wp-helm

        release "wp-helm" uninstalled

- kubectl delete namespace wp-helm

        namespace "wp-helm" deleted

---

Task 8: Clean Up and Reflect

1) Take a final look: `kubectl get all -n capstone`

- kubectl get all -n capstone

        NAME                                     READY   STATUS    RESTARTS   AGE
        pod/mysql-0                              1/1     Running   0          29m
        pod/wordpress-5ff669bb88-nrbl6           1/1     Running   0          31m
        pod/wordpress-5ff669bb88-wrlkn           1/1     Running   0          75m
        pod/wp-helm-mariadb-0                    1/1     Running   0          12m
        pod/wp-helm-wordpress-5b7bdb8cd5-r2pm6   1/1     Running   0          12m
        
        NAME                               TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
        service/mysql                      ClusterIP      None            <none>        3306/TCP                     103m
        service/wordpress                  NodePort       10.96.239.72    <none>        80:30080/TCP                 69m
        service/wp-helm-mariadb            ClusterIP      10.96.204.180   <none>        3306/TCP                     12m
        service/wp-helm-mariadb-headless   ClusterIP      None            <none>        3306/TCP                     12m
        service/wp-helm-wordpress          LoadBalancer   10.96.139.171   <pending>     80:32248/TCP,443:31963/TCP   12m
        
        NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/wordpress           2/2     2            2           75m
        deployment.apps/wp-helm-wordpress   1/1     1            1           12m
        
        NAME                                           DESIRED   CURRENT   READY   AGE
        replicaset.apps/wordpress-5ff669bb88           2         2         2       75m
        replicaset.apps/wp-helm-wordpress-5b7bdb8cd5   1         1         1       12m
        
        NAME                               READY   AGE
        statefulset.apps/mysql             1/1     86m
        statefulset.apps/wp-helm-mariadb   1/1     12m
        
        NAME                                                REFERENCE              TARGETS              MINPODS   MAXPODS   REPLICAS   AGE
        horizontalpodautoscaler.autoscaling/wordpress-hpa   Deployment/wordpress   cpu: <unknown>/50%   2         10        2          21m

2) Count the concepts you used: Namespace, Secret, ConfigMap, PVC, StatefulSet, Headless Service, Deployment, NodePort Service, Resource Limits, Probes, HPA, Helm — twelve concepts in one deployment

        yes

3) Delete the namespace: `kubectl delete namespace capstone`

- kubectl delete namespace capstone

        namespace "capstone" deleted

4) Reset default: `kubectl config set-context --current --namespace=default`

- kubectl config set-context --current --namespace=default

        Context "kind-chetan-cluster" modified.

Verify: Did deleting the namespace remove everything?

        yeah it did
