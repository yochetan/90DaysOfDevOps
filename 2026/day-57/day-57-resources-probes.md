Task 1: Resource Requests and Limits

1) Write a Pod manifest with `resources.requests` (cpu: 100m, memory: 128Mi) and `resources.limits` (cpu: 250m, memory: 256Mi)

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

3) Since requests and limits differ, the QoS class is `Burstable`. If equal, it would be `Guaranteed`. If missing, `BestEffort`.

CPU is in millicores: `100m` = 0.1 CPU. Memory is in mebibytes: `128Mi`.

Requests = guaranteed minimum (scheduler uses this for placement). Limits = maximum allowed (kubelet enforces at runtime).

Verify: What QoS class does your Pod have?
