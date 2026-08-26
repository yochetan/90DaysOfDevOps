Task 1: Recall the Kubernetes Story

Before touching a terminal, write down from memory:

1) Why was Kubernetes created? What problem does it solve that Docker alone cannot? (SOMEWHAT IN MY WORDS)

        It was created to automate scaling and healing, as before it Google was using people to do it manually and they had to re-run the application when they had to scale

2) Who created Kubernetes and what was it inspired by?

        Before kubernetes was created, Google developed an internal system called Borg (later named Omega) to deploy and manage thousands of google applications and services on their cluster.
        In 2014, google introduced Kubernetes an open-source platform written in 'Golang' and later donated to CNCF (Cloud Native Computing Foundation).

3) What does the name "Kubernetes" mean?

        The name Kubernetes comes from a Greek word that means "helmsman" or "pilot".

Do not look anything up yet. Write what you remember from the session, then verify against the official docs.


Task 2: Draw the Kubernetes Architecture

From memory, draw or describe the Kubernetes architecture. Your diagram should include:

Control Plane (Master Node):

* API Server — the front door to the cluster, every command goes through it
* etcd — the database that stores all cluster state
* Scheduler — decides which node a new pod should run on
* Controller Manager — watches the cluster and makes sure the desired state matches reality

Worker Node:

* kubelet — the agent on each node that talks to the API server and manages pods
* kube-proxy — handles networking rules so pods can communicate
* Container Runtime — the engine that actually runs containers (containerd, CRI-O)

After drawing, verify your understanding:

* What happens when you run kubectl apply -f pod.yaml? Trace the request through each component.

        THE POD IS CREATED/CONFIGURED

        kubectl → API Server → etcd → Scheduler → Kubelet → Container Runtime → Container

STEP-BY-STEP:

        - kubectl reads pod.yaml.
        - It sends the Pod definition to the kube-apiserver.
        - API server authenticates and authorizes your request.
        - Admission controllers and API validation check the request.
        - The desired Pod state is stored in etcd.
        - The scheduler notices that the Pod doesn't have a node assigned and selects a suitable worker node.
        - The API server records that node assignment.
        - The kubelet on that worker node notices the Pod assignment.
        - Kubelet asks the container runtime, such as containerd, to create the container.
        - The container starts.
        - Kubelet continuously reports the Pod's status back to the API server.

* What happens if the API server goes down?
        
        THE CONTACT FROM MASTER NODE TO WORKER NODE GOES OFFLINE

        The Kubernetes control plane stops accepting new API requests.

        The kubelets and container runtime can continue running existing containers.

        But Kubernetes loses much of its ability to coordinate and manage the cluster. For example, new scheduling and controller-driven changes cannot proceed normally because the control-plane components depend on the API server.

        Once the API server comes back, normal cluster management can resume.

* What happens if a worker node goes down?

        THE POD OR BASICALLY THE THINGS RUNS ON GOES OFFLINE

Suppose:

        Control Plane
             │
             ├── Worker 1 ✅
             └── Worker 2 ❌

        The kubelet on Worker 2 stops reporting its status.

        The control plane eventually detects that the node is unhealthy.

        If Pods were running on that node, Kubernetes can recreate/reschedule managed workloads on healthy nodes, depending on the workload type and configuration.


Task 3: Install kubectl

kubectl is the CLI tool you will use to talk to your Kubernetes cluster.

Install it:

* Using curl:

        curl.exe -LO "https://dl.k8s.io/release/v1.37.0/bin/windows/amd64/kubectl.exe"

* Append or prepend the kubectl binary folder to your PATH environment variable.

Verify:

* kubectl version --client

        Client Version: v1.36.1
        Kustomize Version: v5.8.1


Task 4: Set Up Your Local Cluster

Choose one of the following. Both give you a fully functional Kubernetes cluster on your machine.

Option A: kind (Kubernetes in Docker)

        # Install kind
        # macOS
        brew install kind
        
        # Linux
        curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
        chmod +x ./kind
        sudo mv ./kind /usr/local/bin/kind
        
        # Create a cluster
        kind create cluster --name devops-cluster
        
        # Verify
        kubectl cluster-info
        kubectl get nodes

Option B: minikube

        # Install minikube
        # macOS
        brew install minikube
        
        # Linux
        curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
        sudo install minikube-linux-amd64 /usr/local/bin/minikube
        
        # Start a cluster
        minikube start
        
        # Verify
        kubectl cluster-info
        kubectl get nodes

Write down: Which one did you choose and why?

Option A: kind (Kubernetes in Docker)

* kubectl cluster-info
        
        Kubernetes control plane is running at https://127.0.0.1:56313
        CoreDNS is running at https://127.0.0.1:56313/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

* kubectl get nodes

        NAME                           STATUS   ROLES           AGE   VERSION
        chetan-cluster-control-plane   Ready    control-plane   28h   v1.37.0-rc.1
        chetan-cluster-worker          Ready    <none>          28h   v1.37.0-rc.1
        chetan-cluster-worker2         Ready    <none>          28h   v1.37.0-rc.1


Task 5: Explore Your Cluster

Now that your cluster is running, explore it:

# See cluster info
kubectl cluster-info

        Kubernetes control plane is running at https://127.0.0.1:56313
        CoreDNS is running at https://127.0.0.1:56313/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

# List all nodes
kubectl get nodes
        
        NAME                           STATUS   ROLES           AGE   VERSION
        chetan-cluster-control-plane   Ready    control-plane   28h   v1.37.0-rc.1
        chetan-cluster-worker          Ready    <none>          28h   v1.37.0-rc.1
        chetan-cluster-worker2         Ready    <none>          28h   v1.37.0-rc.1

# Get detailed info about your node
kubectl describe node <node-name>

        kubectl describe node chetan-cluster-control-plane

# List all namespaces
kubectl get namespaces

        NAME                 STATUS   AGE
        default              Active   28h
        flask-app            Active   27h
        kube-node-lease      Active   28h
        kube-public          Active   28h
        kube-system          Active   28h
        local-path-storage   Active   28h

# See ALL pods running in the cluster (across all namespaces)
kubectl get pods -A

        NAMESPACE            NAME                                                   READY   STATUS    RESTARTS        AGE
        default              nginx-pod                                              1/1     Running   2 (5m22s ago)   27h
        flask-app            flask-app                                              1/1     Running   2 (5m22s ago)   27h
        flask-app            flask-app-deployment-86c7456845-2l2m7                  1/1     Running   2 (5m22s ago)   26h
        flask-app            flask-app-deployment-86c7456845-2x7tr                  1/1     Running   2 (5m22s ago)   26h
        flask-app            flask-app-deployment-86c7456845-86dvh                  1/1     Running   2 (5m22s ago)   26h
        flask-app            flask-app-deployment-86c7456845-9tkl8                  1/1     Running   2 (5m22s ago)   26h
        flask-app            flask-app-deployment-86c7456845-kq9x9                  1/1     Running   2 (5m22s ago)   26h
        flask-app            flask-app-deployment-86c7456845-ljvsv                  1/1     Running   2 (5m22s ago)   26h
        flask-app            flask-app-deployment-86c7456845-v2mhc                  1/1     Running   2 (5m22s ago)   26h
        kube-system          coredns-559f6c778d-ftrmq                               1/1     Running   2 (5m22s ago)   28h
        kube-system          coredns-559f6c778d-pz5xr                               1/1     Running   2 (5m22s ago)   28h
        kube-system          etcd-chetan-cluster-control-plane                      1/1     Running   0               5m15s
        kube-system          kindnet-lg7qp                                          1/1     Running   2 (5m22s ago)   28h
        kube-system          kindnet-m5sn5                                          1/1     Running   2 (5m22s ago)   28h
        kube-system          kindnet-pq675                                          1/1     Running   2 (5m22s ago)   28h
        kube-system          kube-apiserver-chetan-cluster-control-plane            1/1     Running   0               5m15s
        kube-system          kube-controller-manager-chetan-cluster-control-plane   1/1     Running   3 (18m ago)     28h
        kube-system          kube-proxy-dflds                                       1/1     Running   2 (5m22s ago)   28h
        kube-system          kube-proxy-r2vpd                                       1/1     Running   2 (5m22s ago)   28h
        kube-system          kube-proxy-rxrff                                       1/1     Running   2 (5m22s ago)   28h
        kube-system          kube-scheduler-chetan-cluster-control-plane            1/1     Running   3 (18m ago)     28h
        local-path-storage   local-path-provisioner-75f7fc7dc5-w84hk                1/1     Running   4 (4m49s ago)   28h

Look at the pods running in the kube-system namespace:

- kubectl get pods -n kube-system
        
        NAME                                                   READY   STATUS    RESTARTS        AGE
        coredns-559f6c778d-ftrmq                               1/1     Running   2 (6m14s ago)   28h
        coredns-559f6c778d-pz5xr                               1/1     Running   2 (6m14s ago)   28h
        etcd-chetan-cluster-control-plane                      1/1     Running   0               6m7s
        kindnet-lg7qp                                          1/1     Running   2 (6m14s ago)   28h
        kindnet-m5sn5                                          1/1     Running   2 (6m14s ago)   28h
        kindnet-pq675                                          1/1     Running   2 (6m14s ago)   28h
        kube-apiserver-chetan-cluster-control-plane            1/1     Running   0               6m7s
        kube-controller-manager-chetan-cluster-control-plane   1/1     Running   3 (19m ago)     28h
        kube-proxy-dflds                                       1/1     Running   2 (6m14s ago)   28h
        kube-proxy-r2vpd                                       1/1     Running   2 (6m14s ago)   28h
        kube-proxy-rxrff                                       1/1     Running   2 (6m14s ago)   28h
        kube-scheduler-chetan-cluster-control-plane            1/1     Running   3 (19m ago)     28h

You should see pods like etcd, kube-apiserver, kube-scheduler, kube-controller-manager, coredns, and kube-proxy. These are the architecture components you drew in Task 2 — running as pods inside the cluster.

Verify: Can you match each running pod in kube-system to a component in your architecture diagram?


Task 6: Practice Cluster Lifecycle

Build muscle memory with cluster operations:

# Delete your cluster
kind delete cluster --name devops-cluster

        Deleting cluster "devops-cluster" ...
        Deleted nodes: ["devops-cluster-control-plane"]

# Recreate it
kind create cluster --name devops-cluster

        Creating cluster "devops-cluster" ...
         • Ensuring node image (kindest/node:v1.36.1) 🖼  ...
         ✓ Ensuring node image (kindest/node:v1.36.1) 🖼
         • Preparing nodes 📦   ...
         ✓ Preparing nodes 📦 
         • Writing configuration 📜  ...
         ✓ Writing configuration 📜
         • Starting control-plane 🕹️  ...
         ✓ Starting control-plane 🕹️
         • Installing CNI 🔌  ...
         ✓ Installing CNI 🔌
         • Installing StorageClass 💾  ...
         ✓ Installing StorageClass 💾
        Set kubectl context to "kind-devops-cluster"
        You can now use your cluster with:
        
        kubectl cluster-info --context kind-devops-cluster

# Verify it is back
kubectl get nodes
        
        NAME                           STATUS     ROLES           AGE   VERSION
        devops-cluster-control-plane   NotReady   control-plane   14s   v1.36.1

Try these useful commands:

# Check which cluster kubectl is connected to
kubectl config current-context

        kind-devops-cluster

# List all available contexts (clusters)
kubectl config get-contexts

        CURRENT   NAME                  CLUSTER               AUTHINFO              NAMESPACE
                  kind-chetan-cluster   kind-chetan-cluster   kind-chetan-cluster   
        *         kind-devops-cluster   kind-devops-cluster   kind-devops-cluster           

# See the full kubeconfig
kubectl config view

        apiVersion: v1
        clusters:
        - cluster:
            certificate-authority-data: DATA+OMITTED
            server: https://127.0.0.1:56313
          name: kind-chetan-cluster
        - cluster:
            certificate-authority-data: DATA+OMITTED
            server: https://127.0.0.1:63668
          name: kind-devops-cluster
        contexts:
        - context:
            cluster: kind-chetan-cluster
            user: kind-chetan-cluster
          name: kind-chetan-cluster
        - context:
            cluster: kind-devops-cluster
            user: kind-devops-cluster
          name: kind-devops-cluster
        current-context: kind-devops-cluster
        kind: Config
        users:
        - name: kind-chetan-cluster
          user:
            client-certificate-data: DATA+OMITTED
            client-key-data: DATA+OMITTED
        - name: kind-devops-cluster
          user:
            client-certificate-data: DATA+OMITTED
            client-key-data: DATA+OMITTED

Write down: What is a kubeconfig? Where is it stored on your machine?

        kubeconfig is a configuration file that tells kubectl how and where to connect to a Kubernetes cluster.

It contains things like:

        Cluster information — API server address
        User credentials — certificates/tokens used for authentication
        Contexts — which cluster and user kubectl should use

Where is it stored?

On Windows PowerShell:

        C:\Users\<YourUsername>\.kube\config

You can check it with:

- kubectl config view
        
        apiVersion: v1
        clusters:
        - cluster:
            certificate-authority-data: DATA+OMITTED
            server: https://127.0.0.1:56313
          name: kind-chetan-cluster
        - cluster:
            certificate-authority-data: DATA+OMITTED
            server: https://127.0.0.1:58935
          name: kind-devops-cluster
        contexts:
        - context:
            cluster: kind-chetan-cluster
            user: kind-chetan-cluster
          name: kind-chetan-cluster
        - context:
            cluster: kind-devops-cluster
            user: kind-devops-cluster
          name: kind-devops-cluster
        current-context: kind-devops-cluster
        kind: Config
        users:
        - name: kind-chetan-cluster
          user:
            client-certificate-data: DATA+OMITTED
            client-key-data: DATA+OMITTED
        - name: kind-devops-cluster
          user:
            client-certificate-data: DATA+OMITTED
            client-key-data: DATA+OMITTED
