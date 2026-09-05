Task 1: Install Helm

1) Install Helm (brew, curl script, or chocolatey depending on your OS)

        installed using curl

curl.exe -LO https://get.helm.sh/helm-v3.19.0-windows-amd64.zip
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed                                                   
100 17.61M 100 17.61M   0      0  3.16M      0   00:05   00:05          3.46M  

2) Verify with `helm version` and `helm env`

Three core concepts:

- *Chart* — a package of Kubernetes manifest templates

- *Release* — a specific installation of a chart in your cluster

- *Repository* — a collection of charts (like a package repo)

Verify: What version of Helm is installed?
