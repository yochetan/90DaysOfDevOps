Task 1: Install Helm

1) Install Helm (brew, curl script, or chocolatey depending on your OS)

- installed using curl

- curl.exe -LO https://get.helm.sh/helm-v3.19.0-windows-amd64.zip
        
          % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                         Dload  Upload  Total   Spent   Left   Speed                                                   
        100 17.61M 100 17.61M   0      0  3.16M      0   00:05   00:05          3.46M  

- Expand-Archive helm-v3.19.0-windows-amd64.zip -DestinationPath . 

- New-Item -ItemType Directory -Force C:\helm
        
        
            Directory: C:\
        
        
        Mode                 LastWriteTime         Length Name                                                                        
        ----                 -------------         ------ ----                                                                        
        d-----          9/6/2026   3:50 AM                helm            

- Copy-Item .\windows-amd64\helm.exe C:\helm\

- $env:Path += ";C:\helm"
    
2) Verify with `helm version` and `helm env`

- helm version

        version.BuildInfo{Version:"v3.19.0", GitCommit:"3d8990f0836691f0229297773f3524598f46bda6", GitTreeState:"clean", GoVersion:"go1.24.7"}

- helm env

        HELM_BIN="C:\helm\helm.exe"
        HELM_BURST_LIMIT="100"
        HELM_CACHE_HOME="C:\Users\gtmew\AppData\Local\Temp\helm"
        HELM_CONFIG_HOME="C:\Users\gtmew\AppData\Roaming\helm"
        HELM_DATA_HOME="C:\Users\gtmew\AppData\Roaming\helm"
        HELM_DEBUG="false"
        HELM_KUBEAPISERVER=""
        HELM_KUBEASGROUPS=""
        HELM_KUBEASUSER=""
        HELM_KUBECAFILE=""
        HELM_KUBECONTEXT=""
        HELM_KUBEINSECURE_SKIP_TLS_VERIFY="false"
        HELM_KUBETLS_SERVER_NAME=""
        HELM_KUBETOKEN=""
        HELM_MAX_HISTORY="10"
        HELM_NAMESPACE="default"
        HELM_PLUGINS="C:\Users\gtmew\AppData\Roaming\helm\plugins"
        HELM_QPS="0.00"
        HELM_REGISTRY_CONFIG="C:\Users\gtmew\AppData\Roaming\helm\registry\config.json"
        HELM_REPOSITORY_CACHE="C:\Users\gtmew\AppData\Local\Temp\helm\repository"
        HELM_REPOSITORY_CONFIG="C:\Users\gtmew\AppData\Roaming\helm\repositories.yaml"

Three core concepts:

- *Chart* — a package of Kubernetes manifest templates

- *Release* — a specific installation of a chart in your cluster

- *Repository* — a collection of charts (like a package repo)

Verify: What version of Helm is installed?

        v3.19.0

---

Task 2: Add a Repository and Search

1) Add the Bitnami repository: `helm repo add bitnami https://charts.bitnami.com/bitnami`

- helm repo add bitnami https://charts.bitnami.com/bitnami

        "bitnami" has been added to your repositories

2) Update: `helm repo update`

- helm repo update

        Hang tight while we grab the latest from your chart repositories...
        ...Successfully got an update from the "bitnami" chart repository
        Update Complete. ⎈Happy Helming!⎈

3) Search: `helm search repo nginx` and `helm search repo bitnami`

- helm search repo nginx

        NAME                                    CHART VERSION   APP VERSION     DESCRIPTION                                       
        bitnami/nginx                           25.1.10         1.31.5          NGINX Open Source is a web server that can be a...
        bitnami/nginx-ingress-controller        12.0.7          1.13.1          NGINX Ingress Controller is an Ingress controll...
        bitnami/nginx-intel                     2.1.15          0.4.9           DEPRECATED NGINX Open Source for Intel is a lig...

- helm search repo bitnami

Verify: How many charts does Bitnami have?

        25+

---

Task 3: Install a Chart

1) Deploy nginx: `helm install my-nginx bitnami/nginx`

- helm install my-nginx bitnami/nginx

        NAME: my-nginx
        LAST DEPLOYED: Sun Sep  6 03:59:19 2026
        NAMESPACE: default
        STATUS: deployed
        REVISION: 1
        TEST SUITE: None
        NOTES:
        CHART NAME: nginx
        CHART VERSION: 25.1.10
        APP VERSION: 1.31.5

2) Check what was created: `kubectl get all`

- kubectl get all

        NAME                            READY   STATUS    RESTARTS   AGE
        pod/my-nginx-558975b7c8-tqnhj   1/1     Running   0          55s
        
        NAME                 TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
        service/kubernetes   ClusterIP      10.96.0.1      <none>        443/TCP                      134m
        service/my-nginx     LoadBalancer   10.96.121.52   <pending>     80:30432/TCP,443:30204/TCP   55s
        
        NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/my-nginx   1/1     1            1           55s
        
        NAME                                  DESIRED   CURRENT   READY   AGE
        replicaset.apps/my-nginx-558975b7c8   1         1         1       55s

3) Inspect the release: `helm list`, `helm status my-nginx`, `helm get manifest my-nginx`

- helm list

        NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
        my-nginx        default         1               2026-09-06 03:59:19.3646182 +0530 IST   deployed        nginx-25.1.10   1.31.5    

- helm status my-nginx

        NAME: my-nginx
        LAST DEPLOYED: Sun Sep  6 03:59:19 2026
        NAMESPACE: default
        STATUS: deployed
        REVISION: 1
        TEST SUITE: None
        NOTES:
        CHART NAME: nginx
        CHART VERSION: 25.1.10
        APP VERSION: 1.31.5

- helm get manifest my-nginx

        a really big manifest file it is

One command replaced writing a Deployment, Service, and ConfigMap by hand.

        yeah it did.

Verify: How many Pods are running? What Service type was created?

        Pods running : 1
        Service type: LoadBalancer

---

Task 4: Customize with Values

1) View defaults: `helm show values bitnami/nginx`

- helm show values bitnami/nginx

2) Install a custom release with `--set replicaCount=3 --set service.type=NodePort`

- helm install nginx-set bitnami/nginx --set replicaCount=3 --set service.type=NodePort

        NAME: nginx-set
        LAST DEPLOYED: Sun Sep  6 04:08:13 2026
        NAMESPACE: default
        STATUS: deployed
        REVISION: 1
        TEST SUITE: None
        NOTES:
        CHART NAME: nginx
        CHART VERSION: 25.1.10
        APP VERSION: 1.31.5

3) Create a `custom-values.yaml` file with replicaCount, service type, and resource limits

`custom-values.yaml`

        replicaCount: 3
        
        service:
          type: NodePort
        
        resources:
          limits:
            cpu: 500m
            memory: 512Mi

4) Install another release using `-f custom-values.yaml`

- helm install nginx-values bitnami/nginx -f .\custom-values.yaml

        NAME: nginx-values
        LAST DEPLOYED: Sun Sep  6 04:11:12 2026
        NAMESPACE: default
        STATUS: deployed
        REVISION: 1
        TEST SUITE: None
        NOTES:
        CHART NAME: nginx
        CHART VERSION: 25.1.10
        APP VERSION: 1.31.5

5) Check overrides: `helm get values <release-name>`

- helm get values nginx-values

        USER-SUPPLIED VALUES:
        replicaCount: 3
        resources:
          limits:
            cpu: 500m
            memory: 512Mi
        service:
          type: NodePort

Verify: Does the values file release have the correct replicas and service type?

        Yes

---

Task 5: Upgrade and Rollback

1) Upgrade: `helm upgrade my-nginx bitnami/nginx --set replicaCount=5`

- helm upgrade my-nginx bitnami/nginx --set replicaCount=5

        Release "my-nginx" has been upgraded. Happy Helming!
        NAME: my-nginx
        LAST DEPLOYED: Sun Sep  6 04:14:44 2026
        NAMESPACE: default
        STATUS: deployed
        REVISION: 2
        TEST SUITE: None
        NOTES:
        CHART NAME: nginx
        CHART VERSION: 25.1.10
        APP VERSION: 1.31.5


2) Check history: `helm history my-nginx`

- helm history my-nginx

        REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION     
        1               Sun Sep  6 03:59:19 2026        superseded      nginx-25.1.10   1.31.5          Install complete
        2               Sun Sep  6 04:14:44 2026        deployed        nginx-25.1.10   1.31.5          Upgrade complete

3) Rollback: `helm rollback my-nginx 1`

- helm rollback my-nginx 1

        Rollback was a success! Happy Helming!

4) Check history again — rollback creates a new revision (3), not overwriting revision 2

- helm history my-nginx   

        REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION     
        1               Sun Sep  6 03:59:19 2026        superseded      nginx-25.1.10   1.31.5          Install complete
        2               Sun Sep  6 04:14:44 2026        superseded      nginx-25.1.10   1.31.5          Upgrade complete
        3               Sun Sep  6 04:16:11 2026        deployed        nginx-25.1.10   1.31.5          Rollback to 1   

Same concept as Deployment rollouts from Day 52, but at the full stack level.

Verify: How many revisions after the rollback?

        3

---

Task 6: Create Your Own Chart

1) Scaffold: `helm create my-app`

- helm create my-app

        Creating my-app

2) Explore the directory: `Chart.yaml`, `values.yaml`, `templates/deployment.yaml`



3) Look at the Go template syntax in templates: `{{ .Values.replicaCount }}`, `{{ .Chart.Name }}`



4) Edit `values.yaml` — set replicaCount to 3 and image to nginx:1.25

        replicaCount: 3
        
        image:
          repository: nginx
          # This sets the pull policy for images.
          pullPolicy: IfNotPresent
          # Overrides the image tag whose default is the chart appVersion.
          tag: "1.25"

5) Validate: `helm lint my-app`

- helm lint .\my-app

        ==> Linting .\my-app
        [INFO] Chart.yaml: icon is recommended
        
        1 chart(s) linted, 0 chart(s) failed
        
6) Preview: `helm template my-release ./my-app`

- helm template my-release ./my-app

        # Source: my-app/templates/serviceaccount.yaml
        apiVersion: v1
        kind: ServiceAccount
        metadata:
          name: my-release-my-app
          labels:
            helm.sh/chart: my-app-0.1.0
            app.kubernetes.io/name: my-app
            app.kubernetes.io/instance: my-release
            app.kubernetes.io/version: "1.16.0"
            app.kubernetes.io/managed-by: Helm
        automountServiceAccountToken: true
        ---
        # Source: my-app/templates/service.yaml
        apiVersion: v1
        kind: Service
        metadata:
          name: my-release-my-app
          labels:
            helm.sh/chart: my-app-0.1.0
            app.kubernetes.io/name: my-app
            app.kubernetes.io/instance: my-release
            app.kubernetes.io/version: "1.16.0"
            app.kubernetes.io/managed-by: Helm
        spec:
          type: ClusterIP
          ports:
            - port: 80
              targetPort: http
              protocol: TCP
              name: http
          selector:
            app.kubernetes.io/name: my-app
            app.kubernetes.io/instance: my-release
        ---
        # Source: my-app/templates/deployment.yaml
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: my-release-my-app
          labels:
            helm.sh/chart: my-app-0.1.0
            app.kubernetes.io/name: my-app
            app.kubernetes.io/instance: my-release
            app.kubernetes.io/version: "1.16.0"
            app.kubernetes.io/managed-by: Helm
        spec:
          replicas: 3
          selector:
            matchLabels:
              app.kubernetes.io/name: my-app
              app.kubernetes.io/instance: my-release
          template:
            metadata:
              labels:
                helm.sh/chart: my-app-0.1.0
                app.kubernetes.io/name: my-app
                app.kubernetes.io/instance: my-release
                app.kubernetes.io/version: "1.16.0"
                app.kubernetes.io/managed-by: Helm
            spec:
              serviceAccountName: my-release-my-app
              containers:
                - name: my-app
                  image: "nginx:1.25"
                  imagePullPolicy: IfNotPresent
                  ports:
                    - name: http
                      containerPort: 80
                      protocol: TCP
                  livenessProbe:
                    httpGet:
                      path: /
                      port: http
                  readinessProbe:
                    httpGet:
                      path: /
                      port: http
        ---
        # Source: my-app/templates/tests/test-connection.yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: "my-release-my-app-test-connection"
          labels:
            helm.sh/chart: my-app-0.1.0
            app.kubernetes.io/name: my-app
            app.kubernetes.io/instance: my-release
            app.kubernetes.io/version: "1.16.0"
            app.kubernetes.io/managed-by: Helm
          annotations:
            "helm.sh/hook": test
        spec:
          containers:
            - name: wget
              image: busybox
              command: ['wget']
              args: ['my-release-my-app:80']
          restartPolicy: Never

7) Install: `helm install my-release ./my-app`

- helm install my-release .\my-app

        NAME: my-release
        LAST DEPLOYED: Sun Sep  6 04:27:19 2026
        NAMESPACE: default
        STATUS: deployed
        REVISION: 1

8) Upgrade: `helm upgrade my-release ./my-app --set replicaCount=5`

- helm upgrade my-release .\my-app --set replicaCount=5

        Release "my-release" has been upgraded. Happy Helming!
        NAME: my-release
        LAST DEPLOYED: Sun Sep  6 04:27:59 2026
        NAMESPACE: default
        STATUS: deployed
        REVISION: 2

Verify: After installing, 3 replicas? After upgrading, 5?

- kubectl get deployment                                   

        NAME                READY   UP-TO-DATE   AVAILABLE   AGE
        my-release-my-app   5/5     5            5           102s

---

Task 7: Clean Up

1) Uninstall all releases: `helm uninstall <name>` for each

- helm uninstall my-nginx

        release "my-nginx" uninstalled

- helm uninstall nginx-set

        release "nginx-set" uninstalled

- helm uninstall nginx-values

        release "nginx-values" uninstalled

- helm uninstall my-release

        release "my-release" uninstalled

2) Remove chart directory and values file

- Remove-Item -Recurse -Force .\my-app

3) Use `--keep-history` if you want to retain release history for auditing



Verify: Does `helm list` show zero releases?

