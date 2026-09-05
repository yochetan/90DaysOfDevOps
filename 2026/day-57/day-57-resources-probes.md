Task 1: Resource Requests and Limits

1) Write a Pod manifest with `resources.requests` (cpu: 100m, memory: 128Mi) and `resources.limits` (cpu: 250m, memory: 256Mi)

`resource-pod.yml`

        apiVersion: v1
        kind: Pod
        metadata:
          name: resource-pod
        spec:
          containers:
            - name: nginx
              image: nginx:latest
              resources:
                requests:
                  cpu: 100m
                  memory: 128Mi
                limits:
                  cpu: 250m
                  memory: 256Mi

2) Apply and inspect with `kubectl describe pod` — look for the Requests, Limits, and QoS Class sections

- kubectl apply -f resource-pod.yml

        pod/resource-pod created

- kubectl describe pod 

        Name:             resource-pod
        Namespace:        default
        Priority:         0
        Service Account:  default
        Node:             chetan-cluster-worker2/172.18.0.2
        Start Time:       Sat, 05 Sep 2026 17:06:29 +0530
        Labels:           <none>
        Annotations:      <none>
        Status:           Pending
        IP:               
        IPs:              <none>
        Containers:
          nginx:
            Container ID:   
            Image:          nginx:latest
            Image ID:       
            Port:           <none>
            Host Port:      <none>
            State:          Waiting
              Reason:       ContainerCreating
            Ready:          False
            Restart Count:  0
            Limits:
              cpu:     250m
              memory:  256Mi
            Requests:
              cpu:        100m
              memory:     128Mi

        QoS Class:                   Burstable

3) Since requests and limits differ, the QoS class is `Burstable`. If equal, it would be `Guaranteed`. If missing, `BestEffort`.

- Guaranteed → requests and limits are set and equal for every container.
- Burstable → requests/limits are set but don't meet Guaranteed criteria.
- BestEffort → no requests or limits are specified.

CPU is in millicores: `100m` = 0.1 CPU. Memory is in mebibytes: `128Mi`.

Requests = guaranteed minimum (scheduler uses this for placement). Limits = maximum allowed (kubelet enforces at runtime).

Verify: What QoS class does your Pod have?

        Burstable

---

Task 2: OOMKilled — Exceeding Memory Limits

1) Write a Pod manifest using the `polinux/stress` image with a memory limit of `100Mi`

`oom-pod.yml`

        apiVersion: v1
        kind: Pod
        metadata:
          name: oom-pod
        spec:
          containers:
            - name: stress
              image: polinux/stress
              resources:
                limits:
                  memory: 100Mi
              command: ["stress"]
              args: ["--vm", "1", "--vm-bytes", "200M", "--vm-hang", "1"]

2) Set the stress command to allocate 200M of memory: `command: ["stress"] args: ["--vm", "1", "--vm-bytes", "200M", "--vm-hang", "1"]`

           command: ["stress"]
           args: ["--vm", "1", "--vm-bytes", "200M", "--vm-hang", "1"]

3) Apply and watch — the container gets killed immediately

- kubectl apply -f .\oom-pod.yml

        pod/oom-pod created

- kubectl get pod oom-pod -w

        NAME      READY   STATUS      RESTARTS     AGE
        oom-pod   0/1     OOMKilled   1 (2s ago)   9s
        oom-pod   0/1     CrashLoopBackOff   1 (1s ago)   10s
        oom-pod   1/1     Running            2 (14s ago)   23s
        oom-pod   0/1     OOMKilled          2 (14s ago)   23s

CPU is throttled when over limit. Memory is killed — no mercy.

Check `kubectl describe pod` for `Reason: OOMKilled` and `Exit Code: 137` (128 + SIGKILL).

- kubectl describe pod oom-pod
  
        Name:             oom-pod
        Namespace:        default
        Priority:         0
        Service Account:  default
        Node:             chetan-cluster-worker2/172.18.0.2
        Start Time:       Sat, 05 Sep 2026 17:15:57 +0530
        Labels:           <none>
        Annotations:      <none>
        Status:           Running
        IP:               10.244.1.3
        IPs:
          IP:  10.244.1.3
        Containers:
          stress:
            Container ID:  containerd://623c57e7e500f66fc0d052e6cad38a0435a32919f7919894e37dceb5b866acc7
            Image:         polinux/stress
            Image ID:      docker.io/polinux/stress@sha256:b6144f84f9c15dac80deb48d3a646b55c7043ab1d83ea0a697c09097aaad21aa
            Port:          <none>
            Host Port:     <none>
            Command:
              stress
            Args:
              --vm
              1
              --vm-bytes
              200M
              --vm-hang
              1
            State:          Terminated
              Reason:       OOMKilled
              Exit Code:    137
              Started:      Sat, 05 Sep 2026 17:16:20 +0530
              Finished:     Sat, 05 Sep 2026 17:16:20 +0530
            Last State:     Terminated
              Reason:       OOMKilled
              Exit Code:    137
              Started:      Sat, 05 Sep 2026 17:16:05 +0530
              Finished:     Sat, 05 Sep 2026 17:16:06 +0530
            Ready:          False
            Restart Count:  2
            Limits:
              memory:  100Mi
            Requests:
              memory:     100Mi

Verify: What exit code does an OOMKilled container have?

        Exit Code:    137

---

Task 3: Pending Pod — Requesting Too Much

1) Write a Pod manifest requesting `cpu: 100` and `memory: 128Gi`

`pending-pod.yml`

        apiVersion: v1
        kind: Pod
        metadata:
          name: pending-pod
        spec:
          containers:
            - name: nginx
              image: nginx:latest
              resources:
                requests:
                  cpu: "100"
                  memory: "128Gi"

2) Apply and check — STATUS stays `Pending` forever

- kubectl apply -f .\pending-pod.yml

        pod/pending-pod created

- kubectl get pod pending-pod

        NAME          READY   STATUS    RESTARTS   AGE
        pending-pod   0/1     Pending   0          15s

3) Run `kubectl describe pod` and read the Events — the scheduler says exactly why: insufficient resources

- kubectl describe pod pending-pod
        
        Events:
          Type     Reason            Age   From               Message
          ----     ------            ----  ----               -------
          Warning  FailedScheduling  83s   default-scheduler  0/3 nodes are available: 1 node(s) had untolerated taint(s), 2 Insufficient cpu, 2 Insufficient memory. preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling.

Verify: What event message does the scheduler produce?

        Warning  FailedScheduling  83s   default-scheduler  0/3 nodes are available: 1 node(s) had untolerated taint(s), 2 Insufficient cpu, 2 Insufficient memory. preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling.

---

Task 4: Liveness Probe

A liveness probe detects stuck containers. If it fails, Kubernetes restarts the container.

1) Write a Pod manifest with a busybox container that creates `/tmp/healthy` on startup, then deletes it after 30 seconds

`liveness-pod.yml`

        apiVersion: v1
        kind: Pod
        metadata:
          name: liveness-demo
        spec:
          containers:
            - name: busybox
              image: busybox:latest
              command:
                - sh
                - -c
                - |
                  touch /tmp/healthy
                  echo "Health file created"
                  sleep 30
                  rm /tmp/healthy
                  echo "Health file deleted"
                  sleep 3600
              livenessProbe:
                exec:
                  command:
                    - cat
                    - /tmp/healthy
                periodSeconds: 5
                failureThreshold: 3

2) Add a liveness probe using `exec` that runs `cat /tmp/healthy`, with `periodSeconds: 5` and `failureThreshold: 3`

              livenessProbe:
                exec:
                  command:
                    - cat
                    - /tmp/healthy
                periodSeconds: 5
                failureThreshold: 3

3) After the file is deleted, 3 consecutive failures trigger a restart. Watch with `kubectl get pod -w`

- kubectl apply -f liveness-pod.yml 

        pod/liveness-demo created

- kubectl get pod liveness-demo -w

        NAME            READY   STATUS    RESTARTS   AGE
        liveness-demo   1/1     Running   0          14s
        liveness-demo   1/1     Running   1 (1s ago)   77s

- kubectl get pod liveness-demo -o jsonpath="{.status.containerStatuses[0].restartCount}"

        1

- kubectl describe pod liveness-demo

        Name:             liveness-demo
        Namespace:        default
        Priority:         0
        Service Account:  default
        Node:             chetan-cluster-worker2/172.18.0.2
        Start Time:       Sat, 05 Sep 2026 17:46:30 +0530
        Labels:           <none>
        Annotations:      <none>
        Status:           Running
        IP:               10.244.1.4
        IPs:
          IP:  10.244.1.4
        Containers:
          busybox:
            Container ID:  containerd://60431516c75ae358338571416b3a185314f965e1c525ffa75073190ed79384e1
            Image:         busybox:latest
            Image ID:      docker.io/library/busybox@sha256:dc2d74b28e4cf8984fa52af1f39bc7c3d9c73760b41a74d629f5d11b1ab28616
            Port:          <none>
            Host Port:     <none>
            Command:
              sh
              -c
              touch /tmp/healthy
              echo "Health file created"
              sleep 30
              rm /tmp/healthy
              echo "Health file deleted"
              sleep 3600
              
            State:          Running
              Started:      Sat, 05 Sep 2026 17:47:47 +0530
            Last State:     Terminated
              Reason:       Error
              Exit Code:    137
              Started:      Sat, 05 Sep 2026 17:46:36 +0530
              Finished:     Sat, 05 Sep 2026 17:47:46 +0530
            Ready:          True
            Restart Count:  1
            Liveness:       exec [cat /tmp/healthy] delay=0s timeout=1s period=5s #success=1 #failure=3
            Environment:    <none>
            Mounts:
              /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-jdr6b (ro)
        Conditions:
          Type                        Status
          PodReadyToStartContainers   True 
          Initialized                 True 
          Ready                       True 
          ContainersReady             True 
          PodScheduled                True 
        Volumes:
          kube-api-access-jdr6b:
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
          Type     Reason     Age                  From               Message
          ----     ------     ----                 ----               -------
          Normal   Scheduled  2m15s                default-scheduler  Successfully assigned default/liveness-demo to chetan-cluster-worker2
          Normal   Pulled     2m9s                 kubelet            spec.containers{busybox}: Successfully pulled image "busybox:latest" in 4.509s (4.509s including waiting). Image size: 2236931 bytes.
          Normal   Pulling    59s (x2 over 2m14s)  kubelet            spec.containers{busybox}: Pulling image "busybox:latest"
          Normal   Created    58s (x2 over 2m9s)   kubelet            spec.containers{busybox}: Container created
          Normal   Started    58s (x2 over 2m9s)   kubelet            spec.containers{busybox}: Container started
          Normal   Pulled     58s                  kubelet            spec.containers{busybox}: Successfully pulled image "busybox:latest" in 1.251s (1.251s including waiting). Image size: 2236931 bytes.
          Warning  Unhealthy  14s (x6 over 99s)    kubelet            spec.containers{busybox}: Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
          Normal   Killing    14s (x2 over 89s)    kubelet            spec.containers{busybox}: Container busybox failed liveness probe, will be restarted

Verify: How many times has the container restarted?

        1
