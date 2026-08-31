Task 1: See the Problem — Data Lost on Pod Deletion

1) Write a Pod manifest that uses an `emptyDir` volume and writes a timestamped message to `/data/message.txt`

`emptydir-demo.yaml`

        apiVersion: v1
        kind: Pod
        metadata:
          name: emptydir-demo
        spec:
          containers:
            - name: app
              image: busybox:1.36
              command:
                - sh
                - -c
                - |
                  date "+%Y-%m-%d %H:%M:%S" > /data/message.txt
                  echo "Pod started and timestamp written."
                  sleep 3600
              volumeMounts:
                - name: data
                  mountPath: /data
        
          volumes:
            - name: data
              emptyDir: {}

2) Apply it, verify the data exists with `kubectl exec`

- kubectl apply -f emptydir-demo.yaml

        pod/emptydir-demo created

- kubectl exec emptydir-demo -- cat /data/message.txt

        2026-08-31 18:56:08

3) Delete the Pod, recreate it, check the file again — the old message is gone

- kubectl delete pod emptydir-demo

        pod "emptydir-demo" deleted from default namespace

- kubectl apply -f emptydir-demo.yaml

        pod/emptydir-demo created

- kubectl exec emptydir-demo -- cat /data/message.txt

        2026-08-31 18:57:08

Verify: Is the timestamp the same or different after recreation?

        It is different after recreation.

---

Task 2: Create a PersistentVolume (Static Provisioning)

1) Write a PV manifest with `capacity: 1Gi`, `accessModes: ReadWriteOnce`, `persistentVolumeReclaimPolicy: Retain`, and `hostPath` pointing to `/tmp/k8s-pv-data`

`local-pv.yml`

        apiVersion: v1
        kind: PersistentVolume
        metadata:
          name: local-pv
        
        spec:
          capacity:
            storage: 1Gi
          accessModes:
            - ReadWriteOnce
          persistentVolumeReclaimPolicy: Retain
          hostPath:
            path: /tmp/k8s-pv-data

2) Apply it and check `kubectl get pv` — status should be `Available`

- kubectl apply -f .\local-pv.yml             

        persistentvolume/local-pv created

- kubectl get pv

        NAME        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM               STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
        local-pv    1Gi        RWO            Retain           Available                       manual         <unset>                          6s

Access modes to know:

- `ReadWriteOnce (RWO)` — read-write by a single node
- `ReadOnlyMany (ROX)` — read-only by many nodes
- `ReadWriteMany (RWX)` — read-write by many nodes

`hostPath` is fine for learning, not for production.

Verify: What is the STATUS of the PV?
