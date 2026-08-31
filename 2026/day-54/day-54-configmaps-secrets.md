Task 1: Create a ConfigMap from Literals

1) Use kubectl create configmap with --from-literal to create a ConfigMap called app-config with keys APP_ENV=production, APP_DEBUG=false, and APP_PORT=8080

        kubectl create configmap app-config --from-literal=APP_ENV=production --from-literal=APP_DEBUG=false --from-literal=APP_PORT=8080

2) Inspect it with kubectl describe configmap app-config and kubectl get configmap app-config -o yaml



3) Notice the data is stored as plain text — no encoding, no encryption



Verify: Can you see all three key-value pairs?
