Task 1: See the Problem — Data Lost on Pod Deletion

1) Write a Pod manifest that uses an `emptyDir` volume and writes a timestamped message to `/data/message.txt`

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



3) Delete the Pod, recreate it, check the file again — the old message is gone



Verify: Is the timestamp the same or different after recreation?
