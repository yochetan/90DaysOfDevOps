Task 1: Install the Metrics Server

1) Check if it is already running: `kubectl get pods -n kube-system | grep metrics-serve`

        no it's not running

2) If not, install it:

- Minikube: `minikube addons enable metrics-server`

- Kind/kubeadm: apply the official manifest from the metrics-server GitHub releases

- kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

        serviceaccount/metrics-server created
        clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
        clusterrole.rbac.authorization.k8s.io/system:metrics-server created
        rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
        clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
        clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
        service/metrics-server created
        deployment.apps/metrics-server created
        apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created

3) On local clusters, you may need the `--kubelet-insecure-tls` flag (never in production)

- kubectl patch deployment metrics-server -n kube-system \
  --type='json' \
  -p='[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'

        deployment.apps/metrics-server patched


4) Wait 60 seconds, then verify: `kubectl top nodes` and `kubectl top pods -A`

- kubectl top nodes

        NAME                           CPU(cores)   CPU(%)   MEMORY(bytes)   MEMORY(%)   
        chetan-cluster-control-plane   166m         1%       725Mi           9%          
        chetan-cluster-worker          31m          0%       186Mi           2%          
        chetan-cluster-worker2         25m          0%       189Mi           2%  

- kubectl top pods -A

        NAMESPACE            NAME                                                   CPU(cores)   MEMORY(bytes)   
        kube-system          coredns-559f6c778d-ctgbp                               2m           13Mi            
        kube-system          coredns-559f6c778d-lzbbz                               2m           14Mi            
        kube-system          etcd-chetan-cluster-control-plane                      30m          44Mi            
        kube-system          kindnet-767xw                                          1m           9Mi             
        kube-system          kindnet-p8xtv                                          2m           9Mi             
        kube-system          kindnet-sghcw                                          1m           8Mi             
        kube-system          kube-apiserver-chetan-cluster-control-plane            46m          251Mi           
        kube-system          kube-controller-manager-chetan-cluster-control-plane   22m          53Mi            
        kube-system          kube-proxy-6pt9k                                       1m           13Mi            
        kube-system          kube-proxy-hvs7c                                       1m           13Mi            
        kube-system          kube-proxy-tjtg8                                       1m           13Mi            
        kube-system          kube-scheduler-chetan-cluster-control-plane            8m           26Mi            
        kube-system          metrics-server-84c99cb944-bvkfb                        3m           20Mi            
        local-path-storage   local-path-provisioner-75f7fc7dc5-f6lm4                1m           8Mi            

Verify: What is the current CPU and memory usage of your node?

        1%       725Mi
        0%       186Mi
        0%       189Mi

---

Task 2: Explore kubectl top

1) Run `kubectl top nodes`, `kubectl top pods -A`, `kubectl top pods -A --sort-by=cpu`

- kubectl top nodes

        NAME                           CPU(cores)   CPU(%)   MEMORY(bytes)   MEMORY(%)   
        chetan-cluster-control-plane   166m         1%       725Mi           9%          
        chetan-cluster-worker          31m          0%       186Mi           2%          
        chetan-cluster-worker2         25m          0%       189Mi           2%  

- kubectl top pods -A

        NAMESPACE            NAME                                                   CPU(cores)   MEMORY(bytes)   
        kube-system          coredns-559f6c778d-ctgbp                               2m           13Mi            
        kube-system          coredns-559f6c778d-lzbbz                               2m           14Mi            
        kube-system          etcd-chetan-cluster-control-plane                      30m          44Mi            
        kube-system          kindnet-767xw                                          1m           9Mi             
        kube-system          kindnet-p8xtv                                          2m           9Mi             
        kube-system          kindnet-sghcw                                          1m           8Mi             
        kube-system          kube-apiserver-chetan-cluster-control-plane            46m          251Mi           
        kube-system          kube-controller-manager-chetan-cluster-control-plane   22m          53Mi            
        kube-system          kube-proxy-6pt9k                                       1m           13Mi            
        kube-system          kube-proxy-hvs7c                                       1m           13Mi            
        kube-system          kube-proxy-tjtg8                                       1m           13Mi            
        kube-system          kube-scheduler-chetan-cluster-control-plane            8m           26Mi            
        kube-system          metrics-server-84c99cb944-bvkfb                        3m           20Mi            
        local-path-storage   local-path-provisioner-75f7fc7dc5-f6lm4                1m           8Mi            

- kubectl top pods -A --sort-by=cpu

        NAMESPACE            NAME                                                   CPU(cores)   MEMORY(bytes)   
        kube-system          kube-apiserver-chetan-cluster-control-plane            48m          249Mi           
        kube-system          etcd-chetan-cluster-control-plane                      29m          44Mi            
        kube-system          kube-controller-manager-chetan-cluster-control-plane   22m          52Mi            
        kube-system          kube-scheduler-chetan-cluster-control-plane            7m           26Mi            
        kube-system          metrics-server-84c99cb944-bvkfb                        3m           20Mi            
        kube-system          coredns-559f6c778d-lzbbz                               2m           14Mi            
        kube-system          coredns-559f6c778d-ctgbp                               2m           14Mi            
        kube-system          kindnet-sghcw                                          2m           9Mi             
        kube-system          kindnet-767xw                                          1m           8Mi             
        kube-system          kube-proxy-hvs7c                                       1m           13Mi            
        kube-system          kube-proxy-tjtg8                                       1m           13Mi            
        kube-system          kube-proxy-6pt9k                                       1m           13Mi            
        kube-system          kindnet-p8xtv                                          1m           9Mi             
        local-path-storage   local-path-provisioner-75f7fc7dc5-f6lm4                1m           8Mi          

2) `kubectl top` shows real-time usage, not requests or limits — these are different things

3) Data comes from the Metrics Server, which polls kubelets every 15 seconds

Verify: Which pod is using the most CPU right now?

        kube-apiserver-chetan-cluster-control-plane 

---

Task 3: Create a Deployment with CPU Requests

1) Write a Deployment manifest using the `registry.k8s.io/hpa-example` image (a CPU-intensive PHP-Apache server)

`php-apache-deployment.yml`

        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: php-apache
        spec:
          replicas: 1
          selector:
            matchLabels:
              app: php-apache
          template:
            metadata:
              labels:
                app: php-apache
            spec:
              containers:
                - name: php-apache
                  image: registry.k8s.io/hpa-example
                  ports:
                    - containerPort: 80
                  resources:
                    requests:
                      cpu: 200m

- kubectl apply -f php-apache-deployment.yml

        deployment.apps/php-apache created

2) Set `resources.requests.cpu: 200m` — HPA needs this to calculate utilization percentages

                  resources:
                    requests:
                      cpu: 200m

3) Expose it as a Service: `kubectl expose deployment php-apache --port=80`

- kubectl expose deployment php-apache --port=80

        service/php-apache exposed

Without CPU requests, HPA cannot work — this is the most common HPA setup mistake.

Verify: What is the current CPU usage of the Pod?

- kubectl top pod

        NAME                          CPU(cores)   MEMORY(bytes)   
        php-apache-55cd9796db-9bhck   1m           8Mi

---

Task 4: Create an HPA (Imperative)

1) Run: `kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10`

- kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10

        Flag --cpu-percent has been deprecated, Use --cpu with percentage or resource quantity format (e.g., '70%' for utilization or '500m' for milliCPU).
        horizontalpodautoscaler.autoscaling/php-apache autoscaled

2) Check: `kubectl get hpa` and `kubectl describe hpa php-apache`

- kubectl get hpa

        NAME         REFERENCE               TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
        php-apache   Deployment/php-apache   cpu: 0%/50%   1         10        1          24s

- kubectl describe hpa php-apache

        Name:                                                  php-apache
        Namespace:                                             default
        Labels:                                                <none>
        Annotations:                                           <none>
        CreationTimestamp:                                     Sun, 06 Sep 2026 02:56:36 +0530
        Reference:                                             Deployment/php-apache
        Metrics:                                               ( current / target )
          resource cpu on pods  (as a percentage of request):  0% (1m) / 50%
        Min replicas:                                          1
        Max replicas:                                          10
        Deployment pods:                                       1 current / 1 desired
        Conditions:
          Type            Status  Reason               Message
          ----            ------  ------               -------
          AbleToScale     True    ScaleDownStabilized  recent recommendations were higher than current one, applying the highest recent recommendation
          ScalingActive   True    ValidMetricFound     the HPA was able to successfully calculate a replica count from cpu resource utilization (percentage of request)
          ScalingLimited  False   DesiredWithinRange   the desired count is within the acceptable range
        Events:           <none>

3) TARGETS may show `<unknown>` initially — wait 30 seconds for metrics to arrive

        cpu: 0%/50%

This scales up when average CPU exceeds 50% of requests, and down when it drops below.

Verify: What does the TARGETS column show?

        cpu: 0%/50%

---

Task 5: Generate Load and Watch Autoscaling

1) Start a load generator: `kubectl run load-generator --image=busybox:1.36 --restart=Never -- /bin/sh -c "while true; do wget -q -O- http://php-apache; done"`

- kubectl run load-generator --image=busybox:1.36 --restart=Never -- /bin/sh -c "while true; do wget -q -O- http://php-apache; done"

        pod/load-generator created

2) Watch HPA: `kubectl get hpa php-apache --watch`

- kubectl get hpa php-apache --watch

        NAME         REFERENCE               TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
        php-apache   Deployment/php-apache   cpu: 0%/50%   1         10        1          4m16s
        php-apache   Deployment/php-apache   cpu: 141%/50%   1         10        1          4m30s
        php-apache   Deployment/php-apache   cpu: 483%/50%   1         10        3          4m45s
        php-apache   Deployment/php-apache   cpu: 242%/50%   1         10        6          5m
        php-apache   Deployment/php-apache   cpu: 152%/50%   1         10        10         5m15s

3) Over 1-3 minutes, CPU climbs above 50%, replicas increase, CPU stabilizes

        yes

4) Stop the load: `kubectl delete pod load-generator`

- kubectl delete pod load-generator

        pod "load-generator" deleted from default namespace

5) Scale-down is slow (5-minute stabilization window) — you do not need to wait

        yes

Verify: How many replicas did HPA scale to under load?

        10

---

Task 6: Create an HPA from YAML (Declarative)

1) Delete the imperative HPA: `kubectl delete hpa php-apache`

- kubectl delete hpa php-apache

        horizontalpodautoscaler.autoscaling "php-apache" deleted from default namespace

2) Write an HPA manifest using `autoscaling/v2` API with CPU target at 50% utilization

`php-apache-hpa.yml`

        apiVersion: autoscaling/v2
        kind: HorizontalPodAutoscaler
        metadata:
          name: php-apache
        spec:
          scaleTargetRef:
            apiVersion: apps/v1
            kind: Deployment
            name: php-apache
        
          minReplicas: 1
          maxReplicas: 10
        
          metrics:
            - type: Resource
              resource:
                name: cpu
                target:
                  type: Utilization
                  averageUtilization: 50
        
          behavior:
            scaleUp:
              stabilizationWindowSeconds: 0
        
            scaleDown:
              stabilizationWindowSeconds: 300

3) Add a `behavior` section to control scale-up speed (no stabilization) and scale-down speed (300 second window)

          behavior:
            scaleUp:
              stabilizationWindowSeconds: 0
        
            scaleDown:
              stabilizationWindowSeconds: 300

4) Apply and verify with `kubectl describe hpa`

- kubectl apply -f php-apache-hpa.yml 

        horizontalpodautoscaler.autoscaling/php-apache created

`autoscaling/v2` supports multiple metrics and fine-grained scaling behavior that the imperative command cannot configure.

Verify: What does the `behavior` section control?

        The behavior section controls the speed and stability of HPA scaling.
        For this one we had scaleUp and scaleDown.

---

Task 7: Clean Up

Delete the HPA, Service, Deployment, and load-generator pod. Leave the Metrics Server installed.

        yes deleted HPA, Service, Deployment and load-generator pod.
