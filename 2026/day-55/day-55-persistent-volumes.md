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

`pv.yml`

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

- kubectl apply -f .\pv.yml             

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

        Available

---

Task 3: Create a PersistentVolumeClaim

1) Write a PVC manifest requesting `500Mi` of storage with `ReadWriteOnce` access

`pvc.yml`

        apiVersion: v1
        kind: PersistentVolumeClaim
        metadata:
          name: local-pvc
        spec:
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 500Mi

2) Apply it and check both `kubectl get pvc` and `kubectl get pv`

- kubectl apply -f .\pvc.yml

        persistentvolumeclaim/local-pvc created

- kubectl get pv

        NAME        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM               STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
        local-pv    1Gi        RWO            Retain           Bound       default/local-pvc   standard       <unset>                          9s

- kubectl get pvc

        NAME        STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
        local-pvc   Bound    local-pv   1Gi        RWO            standard       <unset>                 12s

3) Both should show `Bound` — Kubernetes matched them by capacity and access mode

        yes.

Verify: What does the VOLUME column in `kubectl get pvc` show?

        local-pv

---

Task 4: Use the PVC in a Pod — Data That Survives

1) Write a Pod manifest that mounts the PVC at `/data` using `persistentVolumeClaim.claimName`

`pvc-pod.yml`

        apiVersion: v1
        kind: Pod
        metadata:
          name: pvc-pod
        spec:
          containers:
            - name: app
              image: busybox:1.36
              command: ["sh", "-c", "sleep 3600"]
              volumeMounts:
                - name: persistent-storage
                  mountPath: /data
        
          volumes:
            - name: persistent-storage
              persistentVolumeClaim:
                claimName: local-pvc

2) Write data to `/data/message.txt`, then delete and recreate the Pod

- kubectl apply -f pvc-pod.yml

        pod/pvc-pod created

- kubectl exec pvc-pod -- sh -c "touch /data/test.txt"

- kubectl exec pvc-pod -- ls -la /data

        total 4
        drwxr-xr-x    2 root     root            60 Aug 31 20:20 .
        drwxr-xr-x    1 root     root          4096 Aug 31 20:15 ..
        -rw-r--r--    1 root     root             0 Aug 31 20:20 test.txt

- kubectl exec pvc-pod -- sh -c "echo 'Message from first Pod' >> /data/message.txt"

- kubectl exec pvc-pod -- cat /data/message.txt

        Message from first Pod

- kubectl delete pod pvc-pod

        pod "pvc-pod" deleted from default namespace

- kubectl apply -f pvc-pod.yml

        pod/pvc-pod created

- kubectl exec pvc-pod -- sh -c "echo 'Message from second Pod' >> /data/message.txt"

3) Check the file — it should contain data from both Pods

- kubectl exec pvc-pod -- cat /data/message.txt

        Message from first Pod
        Message from second Pod

Verify: Does the file contain data from both the first and second Pod?

        yeah it does.

---

Task 5: StorageClasses and Dynamic Provisioning

1) Run `kubectl get storageclass` and `kubectl describe storageclass`

- kubectl get storageclass

        NAME                 PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
        standard (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  2d1h

- kubectl describe storageclass

        Name:            standard
        IsDefaultClass:  Yes
        Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"},"name":"standard"},"provisioner":"rancher.io/local-path","reclaimPolicy":"Delete","volumeBindingMode":"WaitForFirstConsumer"}
        ,storageclass.kubernetes.io/is-default-class=true
        Provisioner:           rancher.io/local-path
        Parameters:            <none>
        AllowVolumeExpansion:  <unset>
        MountOptions:          <none>
        ReclaimPolicy:         Delete
        VolumeBindingMode:     WaitForFirstConsumer
        Events:                <none>

2) Note the provisioner, reclaim policy, and volume binding mode

        Provisioner:           rancher.io/local-path
        ReclaimPolicy:         Delete
        VolumeBindingMode:     WaitForFirstConsumer

3) With dynamic provisioning, developers only create PVCs — the StorageClass handles PV creation automatically

With dynamic provisioning

The developer only creates:

        PVC
         ↓
        StorageClass
         ↓
        Provisioner
         ↓
        PV automatically created
         ↓
        Pod

Kubernetes dynamically creates the PV when the PVC requests a StorageClass.

Verify: What is the default StorageClass in your cluster?

        Standard

---

Task 6: Dynamic Provisioning

1) Write a PVC manifest that includes `storageClassName: standard` (or your cluster's default)

`pvc.yml`

        apiVersion: v1
        kind: PersistentVolumeClaim
        metadata:
          name: dynamic-pvc
        spec:
          accessModes:
            - ReadWriteOnce
          storageClassName: standard
          resources:
            requests:
              storage: 1Gi

2) Apply it — a PV should appear automatically in `kubectl get pv`

- kubectl apply -f pvc.yml

        persistentvolumeclaim/dynamic-pvc created

- kubectl get pv

        NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                 STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
        local-pv                                   1Gi        RWO            Retain           Bound       default/local-pvc     standard       <unset>                          55m
        pvc-2e362556-69c2-4d6c-91dc-e45c3b686344   1Gi        RWO            Delete           Available       default/dynamic-pvc   standard       <unset>                          9s

3) Use this PVC in a Pod, write data, verify it works

`pod.yml`

        apiVersion: v1
        kind: Pod
        metadata:
          name: pvc-demo
        spec:
          containers:
            - name: app
              image: busybox:1.36
              command: ["/bin/sh", "-c"]
              args:
                - |
                  echo "Hello from dynamic PVC" > /data/message.txt
                  sleep 3600
              volumeMounts:
                - name: storage
                  mountPath: /data
        
          volumes:
            - name: storage
              persistentVolumeClaim:
                claimName: dynamic-pvc

- kubectl apply -f pod.yml

        pod/pvc-demo created

- kubectl get pvc

        NAME          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
        dynamic-pvc   Bound    pvc-2e362556-69c2-4d6c-91dc-e45c3b686344   1Gi        RWO            standard       <unset>                 71s
        local-pvc     Bound    local-pv                                   1Gi        RWO            standard       <unset>                 54m

- kubectl get pv

        NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                 STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
        local-pv                                   1Gi        RWO            Retain           Bound       default/local-pvc     standard       <unset>                          55m
        pvc-2e362556-69c2-4d6c-91dc-e45c3b686344   1Gi        RWO            Delete           Bound       default/dynamic-pvc   standard       <unset>                          9s

- kubectl exec pvc-demo -- cat /data/message.txt

        Hello from dynamic PVC

Verify: How many PVs exist now? Which was manual, which was dynamic?

        local-pv was manual and pvc-2e362556-69c2-4d6c-91dc-e45c3b686344 was dynamic.

---

Task 7: Clean Up

1) Delete all pods first

- kubectl delete pods --all

        pod "emptydir-demo" deleted from default namespace
        pod "pvc-demo" deleted from default namespace
        pod "pvc-pod" deleted from default namespace

2) Delete PVCs — check `kubectl get pv` to see what happened

- kubectl delete pvc local-pvc dynamic-pvc

        persistentvolumeclaim "local-pvc" deleted from default namespace
        persistentvolumeclaim "dynamic-pvc" deleted from default namespace

- kubectl get pv

        NAME        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM               STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
        local-pv    1Gi        RWO            Retain           Released    default/local-pvc   standard       <unset>                          64m

3) The dynamic PV is gone (Delete reclaim policy). The manual PV shows Released (Retain policy).

Your two PVs had different reclaim policies:

| PV        | Type    | Reclaim Policy | After PVC deletion    |
|-----------|---------|----------------|-----------------------|
| local-pv  | Manual  | Retain         | Released              |
| pvc-xxxxx | Dynamic | Delete         | Automatically deleted |

Your standard StorageClass showed:

        RECLAIMPOLICY: Delete

4) Delete the remaining PV manually

- kubectl delete pv local-pv

        persistentvolume "local-pv" deleted

Verify: Which PV was auto-deleted and which was retained? Why?

        The dynamic PV was automatically deleted because its StorageClass (standard) uses the Delete reclaim policy. The manually created local-pv was retained and became Released because it was configured with Retain. The retained PV had to be deleted manually.
