Task 1: Install the Metrics Server

1) Check if it is already running: `kubectl get pods -n kube-system | grep metrics-serve`

        no it's not running

2) If not, install it:

- Minikube: `minikube addons enable metrics-server`

- Kind/kubeadm: apply the official manifest from the metrics-server GitHub releases

3) On local clusters, you may need the `--kubelet-insecure-tls` flag (never in production)

4) Wait 60 seconds, then verify: `kubectl top nodes` and `kubectl top pods -A`

Verify: What is the current CPU and memory usage of your node?
