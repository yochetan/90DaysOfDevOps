Task 1: Explore Default Namespaces

Kubernetes comes with built-in namespaces. List them:

- kubectl get namespaces

        NAME                 STATUS   AGE
        default              Active   20h
        kube-node-lease      Active   20h
        kube-public          Active   20h
        kube-system          Active   20h
        local-path-storage   Active   20h

You should see at least:

* default — where your resources go if you do not specify a namespace
* kube-system — Kubernetes internal components (API server, scheduler, etc.)
* kube-public — publicly readable resources
* kube-node-lease — node heartbeat tracking

Check what is running inside kube-system:

- kubectl get pods -n kube-system
        
        NAME                                                   READY   STATUS    RESTARTS        AGE
        coredns-589f44dc88-8w6qc                               1/1     Running   2 (9m17s ago)   20h
        coredns-589f44dc88-mfbzt                               1/1     Running   2 (9m17s ago)   20h
        etcd-devops-cluster-control-plane                      1/1     Running   0               9m7s
        kindnet-7ktkx                                          1/1     Running   2 (9m17s ago)   20h
        kube-apiserver-devops-cluster-control-plane            1/1     Running   0               9m7s
        kube-controller-manager-devops-cluster-control-plane   1/1     Running   2 (9m17s ago)   20h
        kube-proxy-kfxd6                                       1/1     Running   2 (9m17s ago)   20h
        kube-scheduler-devops-cluster-control-plane            1/1     Running   2 (9m17s ago)   20h

These are the control plane components keeping your cluster alive. Do not touch them.

Verify: How many pods are running in kube-system?

        0


Task 2: Create and Use Custom Namespaces

Create two namespaces — one for a development environment and one for staging:

- kubectl create namespace dev

        namespace/dev created

- kubectl create namespace staging

        namespace/staging created

Verify they exist:

- kubectl get namespaces

        NAME                 STATUS   AGE
        default              Active   20h
        dev                  Active   31s
        kube-node-lease      Active   20h
        kube-public          Active   20h
        kube-system          Active   20h
        local-path-storage   Active   20h
        staging              Active   17s

You can also create a namespace from a manifest:

        # namespace.yaml
        apiVersion: v1
        kind: Namespace
        metadata:
          name: production

- kubectl apply -f namespace.yaml

        namespace/production created

Now run a pod in a specific namespace:

- kubectl run nginx-dev --image=nginx:latest -n dev

        pod/nginx-dev created

- kubectl run nginx-staging --image=nginx:latest -n staging

        pod/nginx-staging created

List pods across all namespaces:

- kubectl get pods -A

        NAMESPACE            NAME                                                   READY   STATUS    RESTARTS      AGE
        dev                  nginx-dev                                              1/1     Running   0             98s
        kube-system          coredns-589f44dc88-8w6qc                               1/1     Running   2 (20m ago)   20h
        kube-system          coredns-589f44dc88-mfbzt                               1/1     Running   2 (20m ago)   20h
        kube-system          etcd-devops-cluster-control-plane                      1/1     Running   0             20m
        kube-system          kindnet-7ktkx                                          1/1     Running   2 (20m ago)   20h
        kube-system          kube-apiserver-devops-cluster-control-plane            1/1     Running   0             20m
        kube-system          kube-controller-manager-devops-cluster-control-plane   1/1     Running   2 (20m ago)   20h
        kube-system          kube-proxy-kfxd6                                       1/1     Running   2 (20m ago)   20h
        kube-system          kube-scheduler-devops-cluster-control-plane            1/1     Running   2 (20m ago)   20h
        local-path-storage   local-path-provisioner-855c7b7774-f895s                1/1     Running   4 (20m ago)   20h
        staging              nginx-staging                                          1/1     Running   0             20s

Notice that kubectl get pods without -n only shows the default namespace. You must specify -n <namespace> or use -A to see everything.

Verify: Does kubectl get pods show these pods? What about kubectl get pods -A?

        kubectl get pods shows only default namespace pods 

        kubectl get pods -A shows all namespace pods


Task 3: Create Your First Deployment

A Deployment tells Kubernetes: "I want X replicas of this Pod running at all times." If a Pod crashes, the Deployment controller recreates it automatically.

Create a file nginx-deployment.yaml:

        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: nginx-deployment
          namespace: dev
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
                image: nginx:1.24
                ports:
                - containerPort: 80

Key differences from a standalone Pod:

* `kind: Deployment` instead of `kind: Pod`
* `apiVersion: apps/v1` instead of `v1`
* `replicas: 3` tells Kubernetes to maintain 3 identical pods
* `selector.matchLabels` connects the Deployment to its Pods
* `template` is the Pod template — the Deployment creates Pods using this blueprint

Apply it:

- kubectl apply -f nginx-deployment.yaml

        deployment.apps/nginx-deployment created

Check the result:

- kubectl get deployments -n dev

        NAME               READY   UP-TO-DATE   AVAILABLE   AGE
        nginx-deployment   1/3     3            1           23s

- kubectl get pods -n dev

        NAME                                READY   STATUS    RESTARTS   AGE
        nginx-deployment-68cd4c497b-2vjd8   1/1     Running   0          47s
        nginx-deployment-68cd4c497b-pfqpv   1/1     Running   0          47s
        nginx-deployment-68cd4c497b-vt2dx   1/1     Running   0          47s
        nginx-dev                           1/1     Running   0          10m

You should see 3 pods with names like nginx-deployment-xxxxx-yyyyy.

Verify: What do the READY, UP-TO-DATE, and AVAILABLE columns mean in the deployment output?

        READY → 3/3 means 3 Pods are ready out of the desired 3 Pods.
        UP-TO-DATE → 3 means 3 Pods are running the latest version of the Deployment configuration.
        AVAILABLE → 3 means 3 Pods are currently available to serve traffic.


Task 4: Self-Healing — Delete a Pod and Watch It Come Back

This is the key difference between a Deployment and a standalone Pod.

# List pods

- kubectl get pods -n dev

        NAME                                READY   STATUS    RESTARTS   AGE
        nginx-deployment-68cd4c497b-2vjd8   1/1     Running   0          4m9s
        nginx-deployment-68cd4c497b-pfqpv   1/1     Running   0          4m9s
        nginx-deployment-68cd4c497b-vt2dx   1/1     Running   0          4m9s
        nginx-dev                           1/1     Running   0          14m

# Delete one of the deployment's pods (use an actual pod name from your output)

- kubectl delete pod <pod-name> -n dev

        kubectl delete pod nginx-deployment-68cd4c497b-2vjd8 -n dev

        pod "nginx-deployment-68cd4c497b-2vjd8" deleted from dev namespace

# Immediately check again

- kubectl get pods -n dev
        
        NAME                                READY   STATUS    RESTARTS   AGE
        nginx-deployment-68cd4c497b-pfqpv   1/1     Running   0          5m41s
        nginx-deployment-68cd4c497b-plwn6   1/1     Running   0          48s
        nginx-deployment-68cd4c497b-vt2dx   1/1     Running   0          5m41s
        nginx-dev                           1/1     Running   0          15m

The Deployment controller detects that only 2 of 3 desired replicas exist and immediately creates a new one. The deleted pod 
is replaced within seconds.

Verify: Is the replacement pod's name the same as the one you deleted, or different?

        Different


Task 5: Scale the Deployment

Change the number of replicas:

# Scale up to 5
- kubectl scale deployment nginx-deployment --replicas=5 -n dev

        deployment.apps/nginx-deployment scaled

- kubectl get pods -n dev

        NAME                                READY   STATUS    RESTARTS   AGE
        nginx-deployment-68cd4c497b-pfqpv   1/1     Running   0          9m3s
        nginx-deployment-68cd4c497b-plwn6   1/1     Running   0          4m10s
        nginx-deployment-68cd4c497b-vmzsm   1/1     Running   0          7s
        nginx-deployment-68cd4c497b-vt2dx   1/1     Running   0          9m3s
        nginx-deployment-68cd4c497b-xnldw   1/1     Running   0          7s
        nginx-dev                           1/1     Running   0          19m

# Scale down to 2
- kubectl scale deployment nginx-deployment --replicas=2 -n dev

        deployment.apps/nginx-deployment scaled

- kubectl get pods -n dev

        NAME                                READY   STATUS      RESTARTS   AGE
        nginx-deployment-68cd4c497b-pfqpv   1/1     Running     0          9m30s
        nginx-deployment-68cd4c497b-plwn6   0/1     Completed   0          4m37s
        nginx-deployment-68cd4c497b-vt2dx   1/1     Running     0          9m30s
        nginx-dev                           1/1     Running     0          19m

Watch how Kubernetes creates or terminates pods to match the desired count.

        yeah it does.

You can also scale by editing the manifest — change `replicas: 4` in your YAML file and run `kubectl apply -f nginx-deployment.yaml` again.

- kubectl apply -f nginx-deployment.yaml

        deployment.apps/nginx-deployment configured

- kubectl get pods -n dev               

        NAME                                READY   STATUS    RESTARTS   AGE
        nginx-deployment-68cd4c497b-2g578   1/1     Running   0          1s
        nginx-deployment-68cd4c497b-pfqpv   1/1     Running   0          11m
        nginx-deployment-68cd4c497b-s8j59   1/1     Running   0          1s
        nginx-deployment-68cd4c497b-vt2dx   1/1     Running   0          11m
        nginx-dev                           1/1     Running   0          21m

Verify: When you scaled down from 5 to 2, what happened to the extra pods?

        it terminates or creates on the given count.


Task 6: Rolling Update

Update the Nginx image version to trigger a rolling update:

- kubectl set image deployment/nginx-deployment nginx=nginx:1.25 -n dev

        deployment.apps/nginx-deployment image updated

Watch the rollout in real time:

- kubectl rollout status deployment/nginx-deployment -n dev

        Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 4 new replicas have been updated...
        Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 4 new replicas have been updated...
        Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 4 new replicas have been updated...
        Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 4 new replicas have been updated...
        Waiting for deployment "nginx-deployment" rollout to finish: 3 out of 4 new replicas have been updated...
        Waiting for deployment "nginx-deployment" rollout to finish: 3 out of 4 new replicas have been updated...
        Waiting for deployment "nginx-deployment" rollout to finish: 3 out of 4 new replicas have been updated...
        Waiting for deployment "nginx-deployment" rollout to finish: 3 out of 4 new replicas have been updated...
        Waiting for deployment "nginx-deployment" rollout to finish: 3 out of 4 new replicas have been updated...
        Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
        Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
        Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
        Waiting for deployment "nginx-deployment" rollout to finish: 3 of 4 updated replicas are available...
        Waiting for deployment "nginx-deployment" rollout to finish: 3 of 4 updated replicas are available...
        Waiting for deployment "nginx-deployment" rollout to finish: 3 of 4 updated replicas are available...
        deployment "nginx-deployment" successfully rolled out

Kubernetes replaces pods one by one — old pods are terminated only after new ones are healthy. This means zero downtime.

Check the rollout history:

- kubectl rollout history deployment/nginx-deployment -n dev

        deployment.apps/nginx-deployment 
        REVISION  CHANGE-CAUSE
        1         <none>
        2         <none>

Now roll back to the previous version:

- kubectl rollout undo deployment/nginx-deployment -n dev

        deployment.apps/nginx-deployment rolled back

- kubectl rollout status deployment/nginx-deployment -n dev

        deployment "nginx-deployment" successfully rolled out

Verify the image is back to the previous version:

- kubectl describe deployment nginx-deployment -n dev | grep Image

        Image:         nginx:1.24

Verify: What image version is running after the rollback?

        1.24


Task 7: Clean Up

- kubectl delete deployment nginx-deployment -n dev

        deployment.apps "nginx-deployment" deleted from dev namespace

- kubectl delete pod nginx-dev -n dev

        pod "nginx-dev" deleted from dev namespace

- kubectl delete pod nginx-staging -n staging

        pod "nginx-staging" deleted from staging namespace

- kubectl delete namespace dev staging production

        namespace "dev" deleted
        namespace "staging" deleted
        namespace "production" deleted

Deleting a namespace removes everything inside it. Be very careful with this in production.

- kubectl get namespaces

        NAME                 STATUS   AGE
        default              Active   20h
        kube-node-lease      Active   20h
        kube-public          Active   20h
        kube-system          Active   20h
        local-path-storage   Active   20h

- kubectl get pods -A

        NAMESPACE            NAME                                                   READY   STATUS    RESTARTS      AGE
        kube-system          coredns-589f44dc88-8w6qc                               1/1     Running   2 (54m ago)   20h
        kube-system          coredns-589f44dc88-mfbzt                               1/1     Running   2 (54m ago)   20h
        kube-system          etcd-devops-cluster-control-plane                      1/1     Running   0             54m
        kube-system          kindnet-7ktkx                                          1/1     Running   2 (54m ago)   20h
        kube-system          kube-apiserver-devops-cluster-control-plane            1/1     Running   0             54m
        kube-system          kube-controller-manager-devops-cluster-control-plane   1/1     Running   2 (54m ago)   20h
        kube-system          kube-proxy-kfxd6                                       1/1     Running   2 (54m ago)   20h
        kube-system          kube-scheduler-devops-cluster-control-plane            1/1     Running   2 (54m ago)   20h
        local-path-storage   local-path-provisioner-855c7b7774-f895s                1/1     Running   4 (53m ago)   20h

Verify: Are all your resources gone?

        Nope, only the development is gone and the namespace - dev, staging, production.
