Task 1: Create a ConfigMap from Literals

1) Use `kubectl create configmap` with `--from-literal` to create a ConfigMap called `app-config` with keys `APP_ENV=production`, `APP_DEBUG=false`, and `APP_PORT=8080`

        kubectl create configmap app-config --from-literal=APP_ENV=production --from-literal=APP_DEBUG=false --from-literal=APP_PORT=8080

2) Inspect it with `kubectl describe configmap app-config` and `kubectl get configmap app-config -o yaml`

- kubectl describe configmap app-config

        Name:         app-config
        Namespace:    default
        Labels:       <none>
        Annotations:  <none>
        
        Data
        ====
        APP_DEBUG:
        ----
        false
        
        APP_ENV:
        ----
        production
        
        APP_PORT:
        ----
        8080
        
        
        BinaryData
        ====
        
        Events:  <none>

- kubectl get configmap app-config -o yaml

        apiVersion: v1
        data:
          APP_DEBUG: "false"
          APP_ENV: production
          APP_PORT: "8080"
        kind: ConfigMap
        metadata:
          creationTimestamp: "2026-08-31T07:53:40Z"
          name: app-config
          namespace: default
          resourceVersion: "25751"
          uid: 0319ff2d-03db-4e56-9ff9-9de37817a4ad

3) Notice the data is stored as plain text — no encoding, no encryption

        yes

Verify: Can you see all three key-value pairs?

        yes

---

Task 2: Create a ConfigMap from a File

1) Write a custom Nginx config file that adds a /health endpoint returning "healthy"

- default.conf

        server {
            listen 80;
            server_name _;
        
            location / {
                root /usr/share/nginx/html;
                index index.html;
            }
        
            location /health {
                default_type text/plain;
                return 200 "healthy\n";
            }
        }

2) Create a ConfigMap from this file using `kubectl create configmap nginx-config --from-file=default.conf=<your-file>`

        kubectl create configmap nginx-config --from-file=default.conf=default.conf

3) The key name (default.conf) becomes the filename when mounted into a Pod

`pod.yml`

        kind: Pod
        apiVersion: v1
        
        metadata:
          name: nginx-pod
        
        spec:
          containers:
          - name: nginx
            image: nginx:latest
            ports:
            - containerPort: 80
        
            volumeMounts:
                - name: nginx-config
                  mountPath: /etc/nginx/conf.d/default.conf
                  subPath: default.conf
        
          volumes:
            - name: nginx-config
              configMap:
                name: nginx-config

- kubectl apply -f pod.yml 

        pod/nginx-pod created

- kubectl get configmap nginx-config -o yaml

        apiVersion: v1
        data:
          default.conf: "server {\r\n    listen 80;\r\n    server_name _;\r\n\r\n    location
            / {\r\n        root /usr/share/nginx/html;\r\n        index index.html;\r\n    }\r\n\r\n
            \   location /health {\r\n        default_type text/plain;\r\n        return 200
            \"healthy\\n\";\r\n    }\r\n}"
        kind: ConfigMap
        metadata:
          creationTimestamp: "2026-08-31T08:09:51Z"
          name: nginx-config
          namespace: default
          resourceVersion: "27337"
          uid: 835c1a95-f4ac-42af-b372-ea2feadfd737

- kubectl exec nginx-pod -- cat /etc/nginx/conf.d/default.conf

        server {
            listen 80;
            server_name _;
        
            location / {
                root /usr/share/nginx/html;
                index index.html;
            }
        
            location /health {
                default_type text/plain;
                return 200 "healthy\n";
            }
        }

- kubectl exec nginx-pod -- curl http://localhost/health

          % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                         Dload  Upload   Total   Spent    Left  Speed
        100     8  100     8    0     0   9101      0 --:--:-- --:--:-- --:--:--  8000
        healthy

Verify: Does `kubectl get configmap nginx-config -o yaml` show the file contents?

        yes it does

---

Task 3: Use ConfigMaps in a Pod

1) Write a Pod manifest that uses `envFrom` with `configMapRef` to inject all keys from `app-config` as environment variables. Use a busybox container that prints the values.

`app-config`

        apiVersion: v1
        data:
          APP_DEBUG: "false"
          APP_ENV: production
          APP_PORT: "8080"
        kind: ConfigMap
        metadata:
          creationTimestamp: "2026-08-31T07:53:40Z"
          name: app-config
          namespace: default
          resourceVersion: "25751"
          uid: 0319ff2d-03db-4e56-9ff9-9de37817a4ad

`app-config-pod.yaml`

        apiVersion: v1
        kind: Pod
        metadata:
          name: app-config-pod
        spec:
          containers:
            - name: busybox
              image: busybox:1.36
              command: ["sh", "-c"]
              args:
                - |
                  echo "APP_ENV=$APP_ENV"
                  echo "APP_DEBUG=$APP_DEBUG"
                  echo "APP_PORT=$APP_PORT"
                  sleep 3600
              envFrom:
                - configMapRef:
                    name: app-config

- kubectl logs app-config-pod

        APP_ENV=production
        APP_DEBUG=false
        APP_PORT=8080

2) Write a second Pod manifest that mounts `nginx-config` as a volume at `/etc/nginx/conf.d`. Use the nginx image.

`nginx-config-pod.yaml`
        
        apiVersion: v1
        kind: Pod
        metadata:
          name: nginx-config-pod
        spec:
          containers:
            - name: nginx
              image: nginx:alpine
              ports:
                - containerPort: 80
              volumeMounts:
                - name: nginx-config-volume
                  mountPath: /etc/nginx/conf.d
          volumes:
            - name: nginx-config-volume
              configMap:
                name: nginx-config

- kubectl apply -f nginx-config-pod.yaml

        pod/nginx-config-pod created

- kubectl exec nginx-config-pod -- ls /etc/nginx/conf.d

        default.conf

3) Test that the mounted config works: `kubectl exec <pod> -- curl -s http://localhost/health`

- kubectl exec nginx-config-pod -- curl -s http://localhost/health

        healthy

Use environment variables for simple key-value settings. Use volume mounts for full config files.

| Configuration             | Kubernetes method      | Example                        |
|---------------------------|------------------------|--------------------------------|
| Simple key-value settings | envFrom + configMapRef | APP_ENV=development            |
| Full configuration files  | ConfigMap volume mount | /etc/nginx/conf.d/default.conf |

Verify: Does the /health endpoint respond?

        yeah it did.

---

Task 4: Create a Secret

1) Use `kubectl create secret generic db-credentials` with `--from-literal` to store `DB_USER=admin` and `DB_PASSWORD=s3cureP@ssw0rd`

- kubectl create secret generic db-credentials --from-literal=DB_USER=admin --from-literal=DB_PASSWORD=s3cureP@ssw0rd

        secret/db-credentials created

2) Inspect with `kubectl get secret db-credentials -o yaml` — the values are base64-encoded

- kubectl get secret db-credentials -o yaml

        apiVersion: v1
        data:
          DB_PASSWORD: czNjdXJlUEBzc3cwcmQ=
          DB_USER: YWRtaW4=
        kind: Secret
        metadata:
          creationTimestamp: "2026-08-31T18:08:38Z"
          name: db-credentials
          namespace: default
          resourceVersion: "48797"
          uid: 2afa4169-59c9-4927-80ea-d791c0823bb7
        type: Opaque

3) Decode a value: `echo '<base64-value>' | base64 --decode`

- echo 'YWRtaW4=' | base64 --decode

        admin

- echo 'czNjdXJlUEBzc3cwcmQ=' | base64 --decode

        s3cureP@ssw0rd

*base64 is encoding, not encryption*. Anyone with cluster access can decode Secrets. The real advantages are RBAC separation, tmpfs storage on nodes, and optional encryption at rest.

Verify: Can you decode the password back to plaintext?

        yeah I did.

---

Task 5: Use Secrets in a Pod

1) Write a Pod manifest that injects `DB_USER` as an environment variable using `secretKeyRef`

`secret-pod.yaml`

        apiVersion: v1
        kind: Pod
        metadata:
          name: secret-demo
        spec:
          containers:
            - name: app
              image: busybox:1.36
              command: ["sh", "-c"]
              args:
                - |
                  echo "DB_USER environment variable:"
                  echo "$DB_USER"
                  echo
                  echo "Secret files:"
                  ls -l /etc/db-credentials
                  echo
                  echo "Secret contents:"
                  for file in /etc/db-credentials/*; do
                    echo "$file:"
                    cat "$file"
                    echo
                  done
                  sleep 3600
              env:
                - name: DB_USER
                  valueFrom:
                    secretKeyRef:
                      name: db-credentials
                      key: DB_USER
              volumeMounts:
                - name: db-credentials
                  mountPath: /etc/db-credentials
                  readOnly: true
        
          volumes:
            - name: db-credentials
              secret:
                secretName: db-credentials
        
2) In the same Pod, mount the entire `db-credentials` Secret as a volume at `/etc/db-credentials` with `readOnly: true`

                      volumeMounts:
                        - name: db-credentials
                          mountPath: /etc/db-credentials
                          readOnly: true
                
                  volumes:
                    - name: db-credentials
                      secret:
                        secretName: db-credentials

3) Verify: each Secret key becomes a file, and the content is the decoded plaintext value

- kubectl apply -f  .\secret-pod.yaml 

        pod/secret-demo created

- kubectl logs secret-demo               

        DB_USER environment variable:
        admin
        
        Secret files:
        total 0
        lrwxrwxrwx    1 root     root            18 Aug 31 18:28 DB_PASSWORD -> ..data/DB_PASSWORD
        lrwxrwxrwx    1 root     root            14 Aug 31 18:28 DB_USER -> ..data/DB_USER
        
        Secret contents:
        /etc/db-credentials/DB_PASSWORD:
        s3cureP@ssw0rd
        /etc/db-credentials/DB_USER:
        admin

- kubectl exec secret-demo -- ls -l /etc/db-credentials

        total 0
        lrwxrwxrwx    1 root     root            18 Aug 31 18:28 DB_PASSWORD -> ..data/DB_PASSWORD
        lrwxrwxrwx    1 root     root            14 Aug 31 18:28 DB_USER -> ..data/DB_USER

Verify: Are the mounted file values plaintext or base64?

        plaintext

---

Task 6: Update a ConfigMap and Observe Propagation

1) Create a ConfigMap `live-config` with a key `message=hello`

- kubectl create configmap live-config --from-literal=message=hello

        configmap/live-config created

- kubectl get configmap live-config -o yaml

        apiVersion: v1
        data:
          message: hello
        kind: ConfigMap
        metadata:
          creationTimestamp: "2026-08-31T18:33:56Z"
          name: live-config
          namespace: default
          resourceVersion: "51294"
          uid: d47635eb-78c8-49ac-a677-473f9cff3b1c

2) Write a Pod that mounts this ConfigMap as a volume and reads the file in a loop every 5 seconds

`live-config-pod.yaml`

        apiVersion: v1
        kind: Pod
        metadata:
          name: live-config-pod
        spec:
          containers:
            - name: app
              image: busybox:1.36
              command:
                - /bin/sh
                - -c
                - |
                  while true; do
                    echo "Message: $(cat /etc/config/message)"
                    sleep 5
                  done
              volumeMounts:
                - name: config-volume
                  mountPath: /etc/config
        
          volumes:
            - name: config-volume
              configMap:
                name: live-config

3) Update the ConfigMap: `kubectl patch configmap live-config --type merge -p '{"data":{"message":"world"}}'`

- kubectl patch configmap live-config --type merge -p '{\"data\":{\"message\":\"world\"}}'

        configmap/live-config patched

4) Wait 30-60 seconds — the volume-mounted value updates automatically

- kubectl logs -f live-config-pod

        Message: hello
        Message: hello
        Message: hello
        Message: hello
        Message: world
        Message: world
        Message: world
        Message: world

5) Environment variables from earlier tasks do NOT update — they are set at pod startup only

| ConfigMap usage                    | Updates automatically?              |
|------------------------------------|-------------------------------------|
| Volume mount                       | ✅ Yes                               |
| Environment variable               | ❌ No                                |
| Pod restart after ConfigMap update | Environment variable gets new value |

- ConfigMap → Volume → file → can update while Pod runs

- ConfigMap → Environment variable → fixed when container starts

Verify: Did the volume-mounted value change without a pod restart?

        yeah it did.

---

Task 7: Clean Up

Delete all pods, ConfigMaps, and Secrets you created.

- kubectl delete pods --all

        pod "app-config-pod" deleted from default namespace
        pod "live-config-pod" deleted from default namespace
        pod "nginx-config-pod" deleted from default namespace
        pod "nginx-pod" deleted from default namespace
        pod "secret-demo" deleted from default namespace

- kubectl delete configmaps --all

        configmap "app-config" deleted from default namespace
        configmap "kube-root-ca.crt" deleted from default namespace
        configmap "live-config" deleted from default namespace
        configmap "nginx-config" deleted from default namespace

- kubectl delete secret db-credentials

        secret "db-credentials" deleted from default namespace
