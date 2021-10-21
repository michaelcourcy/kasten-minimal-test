# Kasten cheat sheet for minimal install and test

A list of reccuring operations and tests suite for kasten.

## 1. Install helm

```
HELM_VERSION=v3.7.0
curl -fsSL https://get.helm.sh/helm--${HELM_VERSION}-linux-amd64.tar.gz | tar -zxvf - -C ~/bin/ linux-amd64/helm --strip=1
```

## 2. Add kasten repo 

```
helm repo add kasten https://charts.kasten.io/
helm repo update
```

## 3. Annotate OCS volumlesnasphotclass 

```
oc annotate volumesnapshotclass ocs-storagecluster-cephfsplugin-snapclass k10.kasten.io/is-snapshot-class=true
oc annotate volumesnapshotclass ocs-storagecluster-rbdplugin-snapclass k10.kasten.io/is-snapshot-class=true
```


# 4. Install Kasten on openshift with tokenauth 

```
oc create ns kasten-io
helm install k10 kasten/k10 --namespace=kasten-io \
  --set scc.create=true \
  --set route.enabled=true \
  --set route.tls.enabled=true \
  --set auth.tokenAuth.enabled=true
```

# 5. Create a mysql application with some data in

Create the mysql app.

``` 
kubectl create namespace mysql
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
  namespace: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  serviceName: mysql
  replicas: 1
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0.26
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: ultrasecurepassword   
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:      
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 5Gi
EOF
```

Now create some data 

```
kubectl exec -ti mysql-0 -n mysql -- bash

mysql --user=root --password=ultrasecurepassword
CREATE DATABASE test;
USE test;
CREATE TABLE pets (name VARCHAR(20), owner VARCHAR(20), species VARCHAR(20), sex CHAR(1), birth DATE, death DATE);
INSERT INTO pets VALUES ('Puffball','Diane','hamster','f','1999-03-30',NULL);
SELECT * FROM pets;
+----------+-------+---------+------+------------+-------+
| name     | owner | species | sex  | birth      | death |
+----------+-------+---------+------+------------+-------+
| Puffball | Diane | hamster | f    | 1999-03-30 | NULL  |
+----------+-------+---------+------+------------+-------+
1 row in set (0.00 sec)
```
