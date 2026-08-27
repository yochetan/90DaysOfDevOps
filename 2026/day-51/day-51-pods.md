The Anatomy of a Kubernetes Manifest

Every Kubernetes resource is defined using a YAML manifest with four required top-level fields:

        apiVersion: v1          # Which API version to use
        kind: Pod               # What type of resource
        metadata:               # Name, labels, namespace
          name: my-pod
          labels:
            app: my-app
        spec:                   # The actual specification (what you want)
          containers:
          - name: my-container
            image: nginx:latest
            ports:
            - containerPort: 80

* apiVersion — tells Kubernetes which API group to use. For Pods, it is v1.
* kind — the resource type. Today it is Pod. Later you will use Deployment, Service, etc.
* metadata — the identity of your resource. name is required. labels are key-value pairs used for organization and selection.
* spec — the desired state. For a Pod, this means which containers to run, which images, which ports, etc.


Task 1: Create Your First Pod (Nginx)

Create a file called nginx-pod.yaml:

        apiVersion: v1
        kind: Pod
        metadata:
          name: nginx-pod
          labels:
            app: nginx
        spec:
          containers:
          - name: nginx
            image: nginx:latest
            ports:
            - containerPort: 80

Apply it:

        - kubectl apply -f nginx-pod.yml

        nginx-pod/nginx-pod created

Verify:

        - kubectl get pods

        NAME        READY   STATUS    RESTARTS   AGE
        nginx-pod   1/1     Running   0          82s        

        - kubectl get pods -o wide
        
        NAME        READY   STATUS    RESTARTS   AGE    IP           NODE                           NOMINATED NODE   READINESS GATES
        nginx-pod   1/1     Running   0          2m2s   10.244.0.5   devops-cluster-control-plane   <none>           <none>

Wait until the STATUS shows Running. Then explore:

# Detailed info about the pod
kubectl describe pod nginx-pod

        Name:             nginx-pod
        Namespace:        default
        Priority:         0
        Service Account:  default
        Node:             devops-cluster-control-plane/172.18.0.5
        Start Time:       Thu, 27 Aug 2026 04:52:14 +0530
        Labels:           <none>
        Annotations:      <none>
        Status:           Running
        IP:               10.244.0.5
        IPs:
          IP:  10.244.0.5
        Containers:
          nginx:
            Container ID:   containerd://727bda51b44cea03b65e99b1b3237ffc566724c025ea0adadbb42f3bd340d8c7
            Image:          nginx:latest
            Image ID:       docker.io/library/nginx@sha256:b34848eff6db786b6b1282d3a9c3fd0b5563dfb6d261df4923378b419e0d24f0
            Port:           80/TCP
            Host Port:      0/TCP
            State:          Running
              Started:      Thu, 27 Aug 2026 04:52:39 +0530
            Ready:          True
            Restart Count:  0
            Environment:    <none>
            Mounts:
              /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-5pp2n (ro)
        Conditions:
          Type                        Status
          PodReadyToStartContainers   True 
          Initialized                 True 
          Ready                       True 
          ContainersReady             True 
          PodScheduled                True 
        Volumes:
          kube-api-access-5pp2n:
            Type:                    Projected (a volume that contains injected data from multiple sources)
            TokenExpirationSeconds:  3607
            ConfigMapName:           kube-root-ca.crt
            Optional:                false
            DownwardAPI:             true
        QoS Class:                   BestEffort
        Node-Selectors:              <none>
        Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                                     node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
        Events:
          Type    Reason     Age    From               Message
          ----    ------     ----   ----               -------
          Normal  Scheduled  2m45s  default-scheduler  Successfully assigned default/nginx-pod to devops-cluster-control-plane
          Normal  Pulling    2m44s  kubelet            spec.containers{nginx}: Pulling image "nginx:latest"
          Normal  Pulled     2m20s  kubelet            spec.containers{nginx}: Successfully pulled image "nginx:latest" in 23.907s (23.907s including waiting). Image size: 63348835 bytes.
          Normal  Created    2m20s  kubelet            spec.containers{nginx}: Container created
          Normal  Started    2m20s  kubelet            spec.containers{nginx}: Container started

# Read the logs
kubectl logs nginx-pod

        /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
        /docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
        /docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
        10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
        10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
        /docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
        /docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
        /docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
        /docker-entrypoint.sh: Configuration complete; ready for start up
        2026/08/26 23:22:39 [notice] 1#1: using the "epoll" event method
        2026/08/26 23:22:39 [notice] 1#1: nginx/1.31.4
        2026/08/26 23:22:39 [notice] 1#1: built by gcc 14.2.0 (Debian 14.2.0-19) 
        2026/08/26 23:22:39 [notice] 1#1: OS: Linux 6.18.33.1-microsoft-standard-WSL2
        2026/08/26 23:22:39 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1073741816:1073741816
        2026/08/26 23:22:39 [notice] 1#1: start worker processes
        2026/08/26 23:22:39 [notice] 1#1: start worker process 33
        2026/08/26 23:22:39 [notice] 1#1: start worker process 34
        2026/08/26 23:22:39 [notice] 1#1: start worker process 35
        2026/08/26 23:22:39 [notice] 1#1: start worker process 36
        2026/08/26 23:22:39 [notice] 1#1: start worker process 37
        2026/08/26 23:22:39 [notice] 1#1: start worker process 38
        2026/08/26 23:22:39 [notice] 1#1: start worker process 39
        2026/08/26 23:22:39 [notice] 1#1: start worker process 40
        2026/08/26 23:22:39 [notice] 1#1: start worker process 41
        2026/08/26 23:22:39 [notice] 1#1: start worker process 42
        2026/08/26 23:22:39 [notice] 1#1: start worker process 43
        2026/08/26 23:22:39 [notice] 1#1: start worker process 44

# Get a shell inside the container
kubectl exec -it nginx-pod -- /bin/bash

        root@nginx-pod:/# 

# Inside the container, run:
curl localhost:80

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
        <p>If you see this page, nginx is successfully installed and working.
        Further configuration is required for the web server, reverse proxy, 
        API gateway, load balancer, content cache, or other features.</p>
        
        <p>For online documentation and support please refer to
        <a href="https://nginx.org/">nginx.org</a>.<br/>
        To engage with the community please visit
        <a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
        For enterprise grade support, professional services, additional 
        security features and capabilities please refer to
        <a href="https://f5.com/nginx">f5.com/nginx</a>.</p>
        
        <p><em>Thank you for using nginx.</em></p>
        </body>
        </html>

exit

        exit

Verify: Can you see the Nginx welcome page when you curl from inside the pod?

        yes


Task 2: Create a Custom Pod (BusyBox)

Write a new manifest busybox-pod.yml from scratch (do not copy-paste the nginx one):

        apiVersion: v1
        kind: Pod
        metadata:
          name: busybox-pod
          labels:
            app: busybox
            environment: dev
        spec:
          containers:
          - name: busybox
            image: busybox:latest
            command: ["sh", "-c", "echo Hello from BusyBox && sleep 3600"]

Apply and verify:

- kubectl apply -f busybox-pod.yml
        
        pod/busybox-pod created

- kubectl get pods
        
        NAME          READY   STATUS    RESTARTS   AGE
        busybox-pod   1/1     Running   0          52s

- kubectl logs busybox-pod
        
        Hello from BusyBox

Notice the command field — BusyBox does not run a long-lived server like Nginx. Without a command that keeps it running, the container would exit immediately and the pod would go into CrashLoopBackOff.

Verify: Can you see "Hello from BusyBox" in the logs?
        
        yess


Task 3: Imperative vs Declarative

You have been using the declarative approach (writing YAML, then kubectl apply). Kubernetes also supports imperative commands:

# Create a pod without a YAML file
kubectl run redis-pod --image=redis:latest

        pod/redis-pod created

# Check it
kubectl get pods

        NAME          READY   STATUS              RESTARTS   AGE
        busybox-pod   1/1     Running             0          18m
        nginx-pod     1/1     Running             0          30m
        redis-pod     0/1     ContainerCreating   0          13s

Now extract the YAML that Kubernetes generated:

- kubectl get pod redis-pod -o yaml

        apiVersion: v1
        kind: Pod
        metadata:
          creationTimestamp: "2026-08-26T23:52:43Z"
          generation: 1
          labels:
            run: redis-pod
          name: redis-pod
          namespace: default
          resourceVersion: "3711"
          uid: ef8e1e92-ba75-41d0-8e57-03a9134a2b7e
        spec:
          containers:
          - image: redis:latest
            imagePullPolicy: Always
            name: redis-pod
            resources: {}
            terminationMessagePath: /dev/termination-log
            terminationMessagePolicy: File
            volumeMounts:
            - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
              name: kube-api-access-hk4kd
              readOnly: true
          dnsPolicy: ClusterFirst
          enableServiceLinks: true
          nodeName: devops-cluster-control-plane
          preemptionPolicy: PreemptLowerPriority
          priority: 0
          restartPolicy: Always
          schedulerName: default-scheduler
          securityContext: {}
          serviceAccount: default
          serviceAccountName: default
          terminationGracePeriodSeconds: 30
          tolerations:
          - effect: NoExecute
            key: node.kubernetes.io/not-ready
            operator: Exists
            tolerationSeconds: 300
          - effect: NoExecute
            key: node.kubernetes.io/unreachable
            operator: Exists
            tolerationSeconds: 300
          volumes:
          - name: kube-api-access-hk4kd
            projected:
              defaultMode: 420
              sources:
              - serviceAccountToken:
                  expirationSeconds: 3607
                  path: token
              - configMap:
                  items:
                  - key: ca.crt
                    path: ca.crt
                  name: kube-root-ca.crt
              - downwardAPI:
                  items:
                  - fieldRef:
                      apiVersion: v1
                      fieldPath: metadata.namespace
                    path: namespace
        status:
          conditions:
          - lastProbeTime: null
            lastTransitionTime: "2026-08-26T23:52:43Z"
            observedGeneration: 1
            status: "True"
            type: PodReadyToStartContainers
          - lastProbeTime: null
            lastTransitionTime: "2026-08-26T23:52:43Z"
            observedGeneration: 1
            status: "True"
            type: Initialized
          - lastProbeTime: null
            lastTransitionTime: "2026-08-26T23:52:57Z"
            observedGeneration: 1
            status: "True"
            type: Ready
          - lastProbeTime: null
            lastTransitionTime: "2026-08-26T23:52:57Z"
            observedGeneration: 1
            status: "True"
            type: ContainersReady
          - lastProbeTime: null
            lastTransitionTime: "2026-08-26T23:52:43Z"
            observedGeneration: 1
            status: "True"
            type: PodScheduled
          containerStatuses:
          - containerID: containerd://1640408a83ef4c669007ee0a52a0c858e1eb9dbb4a35cd4d0a09f2c900e48a25
            image: docker.io/library/redis:latest
            imageID: docker.io/library/redis@sha256:9c5c616b6e72ea6e38125bb9b12eac2dcd92cd57f8f811ee2fb965ae4cb0f16e
            lastState: {}
            name: redis-pod
            ready: true
            resources: {}
            restartCount: 0
            started: true
            state:
              running:
                startedAt: "2026-08-26T23:52:57Z"
            user:
              linux:
                gid: 0
                supplementalGroups:
                - 0
                uid: 0
            volumeMounts:
            - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
              name: kube-api-access-hk4kd
              readOnly: true
              recursiveReadOnly: Disabled
          hostIP: 172.18.0.5
          hostIPs:
          - ip: 172.18.0.5
          observedGeneration: 1
          phase: Running
          podIP: 10.244.0.7
          podIPs:
          - ip: 10.244.0.7
          qosClass: BestEffort
          resources: {}
          startTime: "2026-08-26T23:52:43Z"

Compare this output with your hand-written manifests. Notice how much extra metadata Kubernetes adds automatically (status, timestamps, uid, resource version).
        
        yeah FR!

You can also use dry-run to generate YAML without creating anything:

- kubectl run test-pod --image=nginx --dry-run=client -o yaml

        apiVersion: v1
        kind: Pod
        metadata:
          labels:
            run: test-pod
          name: test-pod
        spec:
          containers:
          - image: nginx
            name: test-pod
            resources: {}
          dnsPolicy: ClusterFirst
          restartPolicy: Always
        status: {}

This is a powerful trick — use it to quickly scaffold a manifest, then customize it.

Verify: Save the dry-run output to a file and compare its structure with your nginx-pod.yaml. What fields are the same? What is different?

        resources: {}
          dnsPolicy: ClusterFirst
          restartPolicy: Always
        status: {}
        
        these are extra and besides these everything is same.


Task 4: Validate Before Applying

Before applying a manifest, you can validate it:

# Check if the YAML is valid without actually creating the resource
kubectl apply -f nginx-pod.yml --dry-run=client

        nginx-pod/nginx-pod unchanged (dry run)

# Validate against the cluster's API (server-side validation)
kubectl apply -f nginx-pod.yml --dry-run=server

        pod/nginx-pod unchanged (server dry run)

Now intentionally break your YAML (remove the image field or add an invalid field) and run dry-run again. See what error you get.

kubectl apply --dry-run=client -f pod.yml
        
        pod/nginx-pod configured (dry run)

kubectl apply --dry-run=server -f pod.yml 

        The Pod "nginx-pod" is invalid: spec.containers[0].image: Required value

Verify: What error does Kubernetes give when the image field is missing?

        The Pod "nginx-pod" is invalid: spec.containers[0].image: Required value


Task 5: Pod Labels and Filtering

Labels are how Kubernetes organizes and selects resources. You added labels in your manifests — now use them:

# List all pods with their labels
kubectl get pods --show-labels

        NAME          READY   STATUS    RESTARTS      AGE   LABELS
        busybox-pod   1/1     Running   8 (54m ago)   12h   app=busybox,environment=dev
        nginx-pod     1/1     Running   1 (54m ago)   12h   <none>
        redis-pod     1/1     Running   1 (54m ago)   11h   run=redis-pod

# Filter pods by label
kubectl get pods -l app=nginx

        No resources found in default namespace.

kubectl get pods -l environment=dev

        NAME          READY   STATUS    RESTARTS      AGE
        busybox-pod   1/1     Running   9 (19m ago)   12h

# Add a label to an existing pod
kubectl label pod nginx-pod environment=production

        pod/nginx-pod labeled

# Verify
kubectl get pods --show-labels

        NAME          READY   STATUS    RESTARTS      AGE   LABELS
        busybox-pod   1/1     Running   9 (20m ago)   12h   app=busybox,environment=dev
        nginx-pod     1/1     Running   1 (80m ago)   12h   environment=production
        redis-pod     1/1     Running   1 (80m ago)   12h   run=redis-pod

# Remove a label
kubectl label pod nginx-pod environment-

        pod/nginx-pod unlabeled
        
Write a manifest for a third pod with at least 3 labels (app, environment, team). Apply it and practice filtering.


Task 6: Clean Up

Delete all the pods you created:

# Delete by name
kubectl delete pod nginx-pod
kubectl delete pod busybox-pod
kubectl delete pod redis-pod

# Or delete using the manifest file
kubectl delete -f nginx-pod.yaml

# Verify everything is gone
kubectl get pods

Notice that when you delete a standalone Pod, it is gone forever. There is no controller to recreate it. This is why in production you use Deployments (coming on Day 52) instead of bare Pods.
