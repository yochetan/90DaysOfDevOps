Task 1: Explore Default Namespaces

Kubernetes comes with built-in namespaces. List them:

1) kubectl get namespaces

        NAME                 STATUS   AGE
        default              Active   20h
        kube-node-lease      Active   20h
        kube-public          Active   20h
        kube-system          Active   20h
        local-path-storage   Active   20h

You should see at least:

default — where your resources go if you do not specify a namespace
kube-system — Kubernetes internal components (API server, scheduler, etc.)
kube-public — publicly readable resources
kube-node-lease — node heartbeat tracking

Check what is running inside kube-system:

kubectl get pods -n kube-system

These are the control plane components keeping your cluster alive. Do not touch them.

Verify: How many pods are running in kube-system?

