Task 1: Create the Namespace (Day 52)

1) Create a `capstone` namespace

        apiVersion: v1
        kind: Namespace
        metadata:
          name: capstone

2) Set it as your default: `kubectl config set-context --current --namespace=capstone`
