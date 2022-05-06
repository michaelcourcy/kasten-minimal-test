# Create a basic app that use a NFS PVC 

Change the  `#####REPLACE######`place holder before execution 

```
oc create ns kasten-io
cat <<EOF | oc create -f - 
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs-kasten
spec:
  claimRef:
    name: pvc-nfs-kasten
    namespace: kasten-io
  capacity:
    storage: 50Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path:  #####REPLACE######
    server: #####REPLACE######
    readOnly: false
  mountOptions:
      - hard
      - nfsvers=4.1
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nfs-kasten
  namespace: kasten-io
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 50Gi
---
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: alpine
  name: alpine
spec:  
  containers:
  - args:
    - tail
    - -f
    - /dev/null
    volumeMounts:
    - name: data
      mountPath: /data
    image:  alpine
    name: alpine
    resources: {}
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: pvc-nfs-kasten
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
EOF
```

## Test 

Then test you are able to create a file in the nfs share 

```
oc rsh -n kasten-io alpine
echo "hello nfs from a pod" >  /data/hello.txt
```


