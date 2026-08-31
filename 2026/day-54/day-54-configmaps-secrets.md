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

1) Write a Pod manifest that uses envFrom with configMapRef to inject all keys from app-config as environment variables. Use a busybox container that prints the values.

2) Write a second Pod manifest that mounts nginx-config as a volume at /etc/nginx/conf.d. Use the nginx image.

3) Test that the mounted config works: kubectl exec <pod> -- curl -s http://localhost/health

Use environment variables for simple key-value settings. Use volume mounts for full config files.

Verify: Does the /health endpoint respond?

