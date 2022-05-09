# Create a nfs server in kubernetes 

Sometime you don't have nfs share available, for a POC it could be useful to just emulate it.

## Create the nfs sever 

```
kubectl create -f nfs-server.yaml 
```

That will create a nfs server deployment and a nfs-server service.



## Create the nfs pv and pvc 

Grab the ip of the nfs-server service 

```
kubectl get svc nfs-server -n kasten-io
```

For instance you get this output 
```
kubectl get svc nfs-server -n kasten-io
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)            AGE
nfs-server   ClusterIP   172.30.232.83   <none>        2049/TCP,111/UDP   3m17s
```

Use the ip 172.30.232.83 to update the nfs-pv-pvc-test.yaml file line 17 

```
  nfs:
    path: /
    server: 172.30.232.83
    readOnly: false
```

Now create the nfs pv and pvc 
```
kubectl create -f nfs-pv-pvc-test.yaml
```

## test it 

You can now try the [test-connectivity](../test-nfs-connectivity.md) test 
