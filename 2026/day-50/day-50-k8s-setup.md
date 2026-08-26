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

kubectl version --client

        Client Version: v1.36.1
        Kustomize Version: v5.8.1

