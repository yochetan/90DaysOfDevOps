Task 1: Deploy the Application

First, create a Deployment that you will expose with Services. Create `app-deployment.yaml`:

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

- kubectl apply -f app-deployment.yaml

        deployment.apps/web-app created

- kubectl get pods -o wide

        NAME                       READY   STATUS    RESTARTS   AGE   IP            NODE                           NOMINATED NODE   READINESS GATES
        web-app-5c44989c65-8hkxr   1/1     Running   0          5s    10.244.0.27   devops-cluster-control-plane   <none>           <none>
        web-app-5c44989c65-hghvl   1/1     Running   0          5s    10.244.0.28   devops-cluster-control-plane   <none>           <none>
        web-app-5c44989c65-m6dpc   1/1     Running   0          5s    10.244.0.29   devops-cluster-control-plane   <none>           <none>
 
Note the individual Pod IPs. These will change if pods restart — that is the problem Services fix.

Verify: Are all 3 pods running? Note down their IP addresses.

        10.244.0.27
        10.244.0.28
        10.244.0.29


Task 2: ClusterIP Service (Internal Access)

ClusterIP is the default Service type. It gives your Pods a stable internal IP that is only reachable from within the cluster.

Create `clusterip-service.yaml`:
        
        apiVersion: v1
        kind: Service
        metadata:
          name: web-app-clusterip
        spec:
          type: ClusterIP
          selector:
            app: web-app
          ports:
          - port: 80
            targetPort: 80

Key fields:

* `selector.app: web-app` — this Service routes traffic to all Pods with the label app: web-app
* `port: 80` — the port the Service listens on
* `targetPort: 80` — the port on the Pod to forward traffic to

- kubectl apply -f clusterip-service.yaml

        service/web-app-clusterip created

- kubectl get services

        NAME                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
        kubernetes          ClusterIP   10.96.0.1      <none>        443/TCP   21h
        web-app-clusterip   ClusterIP   10.96.254.55   <none>        80/TCP    5s

You should see `web-app-clusterip` with a CLUSTER-IP address. This IP is stable — it will not change even if Pods restart.

Now test it from inside the cluster:

# Run a temporary pod to test connectivity

- kubectl run test-client --image=busybox:latest --rm -it --restart=Never -- sh

        All commands and output from this session will be recorded in container logs, including credentials and sensitive information passed through the command prompt.
        If you don't see a command prompt, try pressing enter.
        / #

# Inside the test pod, run:

- wget -qO- http://web-app-clusterip

        <!DOCTYPE html>
        <html>
        <head>
        <title>Welcome to nginx!</title>
        <style>
        html { color-scheme: light dark; }
        body { width: 35em; margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif; }
        </style>
        </head>
        <body>
        <h1>Welcome to nginx!</h1>
        <p>If you see this page, the nginx web server is successfully installed and
        working. Further configuration is required.</p>
        
        <p>For online documentation and support please refer to
        <a href="http://nginx.org/">nginx.org</a>.<br/>
        Commercial support is available at
        <a href="http://nginx.com/">nginx.com</a>.</p>
        
        <p><em>Thank you for using nginx.</em></p>
        </body>
        </html>

- exit

You should see the Nginx welcome page. The Service load-balanced your request to one of the 3 Pods.

Verify: Does the Service respond? Try running the wget command multiple times — the Service distributes traffic across all healthy Pods.

        yeah it does.


Task 3: Discover Services with DNS

Kubernetes has a built-in DNS server. Every Service gets a DNS entry automatically:

        <service-name>.<namespace>.svc.cluster.local

Test this:

- kubectl run dns-test --image=busybox:latest --rm -it --restart=Never -- sh

        All commands and output from this session will be recorded in container logs, including credentials and sensitive information passed through the command prompt.
        If you don't see a command prompt, try pressing enter.
        / #

# Inside the pod:
# Short name (works within the same namespace)

- wget -qO- http://web-app-clusterip

        <!DOCTYPE html>
        <html>
        <head>
        <title>Welcome to nginx!</title>
        <style>
        html { color-scheme: light dark; }
        body { width: 35em; margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif; }
        </style>
        </head>
        <body>
        <h1>Welcome to nginx!</h1>
        <p>If you see this page, the nginx web server is successfully installed and
        working. Further configuration is required.</p>
        
        <p>For online documentation and support please refer to
        <a href="http://nginx.org/">nginx.org</a>.<br/>
        Commercial support is available at
        <a href="http://nginx.com/">nginx.com</a>.</p>
        
        <p><em>Thank you for using nginx.</em></p>
        </body>
        </html>

# Full DNS name

- wget -qO- http://web-app-clusterip.default.svc.cluster.local

        <!DOCTYPE html>
        <html>
        <head>
        <title>Welcome to nginx!</title>
        <style>
        html { color-scheme: light dark; }
        body { width: 35em; margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif; }
        </style>
        </head>
        <body>
        <h1>Welcome to nginx!</h1>
        <p>If you see this page, the nginx web server is successfully installed and
        working. Further configuration is required.</p>
        
        <p>For online documentation and support please refer to
        <a href="http://nginx.org/">nginx.org</a>.<br/>
        Commercial support is available at
        <a href="http://nginx.com/">nginx.com</a>.</p>
        
        <p><em>Thank you for using nginx.</em></p>
        </body>
        </html>

# Look up the DNS entry

- nslookup web-app-clusterip

        Server:         10.96.0.10
        Address:        10.96.0.10:53
        
        ** server can't find web-app-clusterip.cluster.local: NXDOMAIN
        
        ** server can't find web-app-clusterip.svc.cluster.local: NXDOMAIN
        
        ** server can't find web-app-clusterip.cluster.local: NXDOMAIN
        
        
        ** server can't find web-app-clusterip.svc.cluster.local: NXDOMAIN
        
        Name:   web-app-clusterip.default.svc.cluster.local
        Address: 10.96.254.55

- exit

        Session ended, resume using 'kubectl attach dns-test -c dns-test -n default -i -t' command
        pod "dns-test" deleted from default namespace
        pod default/dns-test terminated (Error)

Both the short name and the full DNS name resolve to the same ClusterIP. In practice, you use the short name when communicating within the same namespace and the full name when reaching across namespaces.

Verify: What IP does nslookup return? Does it match the CLUSTER-IP from kubectl get services?

        Yes, it does.


Task 4: NodePort Service (External Access via Node)

A NodePort Service exposes your application on a port on every node in the cluster. This lets you access the Service from outside the cluster.

Create `nodeport-service.yaml`:
        
        apiVersion: v1
        kind: Service
        metadata:
          name: web-app-nodeport
        spec:
          type: NodePort
          selector:
            app: web-app
          ports:
          - port: 80
            targetPort: 80
            nodePort: 30080

* `nodePort: 30080` — the port opened on every node (must be in range 30000-32767)
* Traffic flow: `<NodeIP>:30080` -> Service -> Pod:80

- kubectl apply -f nodeport-service.yaml

        service/web-app-nodeport created

- kubectl get services
        
        NAME                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
        kubernetes          ClusterIP   10.96.0.1      <none>        443/TCP        21h
        web-app-clusterip   ClusterIP   10.96.254.55   <none>        80/TCP         13m
        web-app-nodeport    NodePort    10.96.143.80   <none>        80:30080/TCP   13s

Access the service:

        # If using Minikube
        minikube service web-app-nodeport --url
        
        # If using Kind, get the node IP first
        kubectl get nodes -o wide
        # Then curl <node-internal-ip>:30080
        
        # If using Docker Desktop
        curl http://localhost:30080

Verify: Can you see the Nginx welcome page from your browser or terminal using the NodePort?

        yes.


Task 5: LoadBalancer Service (Cloud External Access)

In a cloud environment (AWS, GCP, Azure), a LoadBalancer Service provisions a real external load balancer that routes traffic to your nodes.

Create `loadbalancer-service.yaml`:

        apiVersion: v1
        kind: Service
        metadata:
          name: web-app-loadbalancer
        spec:
          type: LoadBalancer
          selector:
            app: web-app
          ports:
          - port: 80
            targetPort: 80

- kubectl apply -f loadbalancer-service.yaml

        service/web-app-loadbalancer created

- kubectl get services

        NAME                   TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
        kubernetes             ClusterIP      10.96.0.1      <none>        443/TCP        21h
        web-app-clusterip      ClusterIP      10.96.254.55   <none>        80/TCP         26m
        web-app-loadbalancer   LoadBalancer   10.96.173.50   <pending>     80:32477/TCP   4s
        web-app-nodeport       NodePort       10.96.143.80   <none>        80:30080/TCP   12m

On a local cluster (Minikube, Kind, Docker Desktop), the EXTERNAL-IP will show `<pending>` because there is no cloud provider to create a real load balancer. This is expected.

If you are using Minikube:

        # Minikube can simulate a LoadBalancer
        minikube tunnel
        # In another terminal, check again:
        kubectl get services

In a real cloud cluster, the EXTERNAL-IP would be a public IP address or hostname provisioned by the cloud provider.

Verify: What does the EXTERNAL-IP column show? Why is it <pending> on a local cluster?

        The EXTERNAL-IP column in kubectl get svc shows the external IP address through which a Kubernetes Service can be accessed from outside the cluster.

        On a local cluster like Kind, there is usually no cloud provider (AWS, Azure, GCP) to automatically provision a load balancer and assign a public/external IP.
        
        So Kubernetes keeps showing:
        
        - EXTERNAL-IP   <pending>
        
        because nothing is available to provide that external IP.


Task 6: Understand the Service Types Side by Side

Check all three services:

- kubectl get services -o wide

        NAME                   TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE     SELECTOR
        kubernetes             ClusterIP      10.96.0.1      <none>        443/TCP        21h     <none>
        web-app-clusterip      ClusterIP      10.96.254.55   <none>        80/TCP         30m     app=web-app
        web-app-loadbalancer   LoadBalancer   10.96.173.50   <pending>     80:32477/TCP   4m45s   app=web-app
        web-app-nodeport       NodePort       10.96.143.80   <none>        80:30080/TCP   17m     app=web-app

Compare them:

| Type         | Accessible From                 | Use Case                                 |
|--------------|---------------------------------|------------------------------------------|
| ClusterIP    | Inside the cluster only         | Internal communication between services  |
| NodePort     | Outside via <NodeIP>:<NodePort> | Development, testing, direct node access |
| LoadBalancer | Outside via cloud load balancer | Production traffic in cloud environments |


Each type builds on the previous one:

* LoadBalancer creates a NodePort, which creates a ClusterIP
* So a LoadBalancer service also has a ClusterIP and a NodePort

Verify this:

- kubectl describe service web-app-loadbalancer

        Name:                     web-app-loadbalancer
        Namespace:                default
        Labels:                   <none>
        Annotations:              <none>
        Selector:                 app=web-app
        Type:                     LoadBalancer
        IP Family Policy:         SingleStack
        IP Families:              IPv4
        IP:                       10.96.173.50
        IPs:                      10.96.173.50
        Port:                     <unset>  80/TCP
        TargetPort:               80/TCP
        NodePort:                 <unset>  32477/TCP
        Endpoints:                10.244.0.29:80,10.244.0.28:80,10.244.0.27:80
        Session Affinity:         None
        External Traffic Policy:  Cluster
        Internal Traffic Policy:  Cluster
        Events:                   <none>

You should see all three: a ClusterIP, a NodePort, and the LoadBalancer configuration.

Verify: Does the LoadBalancer service also have a ClusterIP and NodePort assigned?

        Yes. A Kubernetes LoadBalancer Service normally gets both a ClusterIP and a NodePort in addition to its LoadBalancer configuration.


Task 7: Clean Up

- kubectl delete -f app-deployment.yaml

        deployment.apps "web-app" deleted from default namespace

- kubectl delete -f clusterip-service.yaml

        service "web-app-clusterip" deleted from default namespace

- kubectl delete -f nodeport-service.yaml

        service "web-app-nodeport" deleted from default namespace

- kubectl delete -f loadbalancer-service.yaml

        service "web-app-loadbalancer" deleted from default namespace

- kubectl get pods

        No resources found in default namespace.

- kubectl get services

        NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
        kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   21h

Only the built-in kubernetes service in the default namespace should remain.

Verify: Is everything cleaned up?

        yes.
