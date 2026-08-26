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
* What happens if the API server goes down?
* What happens if a worker node goes down?
