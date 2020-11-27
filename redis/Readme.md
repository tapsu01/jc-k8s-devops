# Redis

**Add bitnami repo**

```none
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install redis bitnami/redis --namespace={namespace}
```

NOTES: Please be patient while the chart is being deployed

Redis can be accessed via port 6379 on the following DNS names from within your cluster:

redis-master.{namespace}.svc.cluster.local for read/write operations
redis-slave.{namespace}.svc.cluster.local for read-only operations

To get your password run:

```none
export REDIS_PASSWORD=$(kubectl get secret --namespace {namespace} redis -o jsonpath="{.data.redis-password}" | base64 --decode)
```

next, run command for password base64

```none
echo -n <password> | base64
```

**or**

Run command only:

```none
kubectl get secret --namespace {namespace} redis -o jsonpath="{.data.redis-password}
```

finally, set redis password to helm-charts/values.yaml

**To connect to your Redis server:**

1. Run a Redis pod that you can use as a client:

```none
kubectl run --namespace {namespace} redis-client --rm --tty -i --restart='Never' \
  --env REDIS_PASSWORD=$REDIS_PASSWORD \
  --image docker.io/bitnami/redis:6.0.3-debian-10-r2 -- bash
```

2. Connect using the Redis CLI:

```none
redis-cli -h redis-master -a $REDIS_PASSWORD
redis-cli -h redis-slave -a $REDIS_PASSWORD
```

To connect to your database from outside the cluster execute the following commands:

```none
kubectl port-forward --namespace jetcare svc/redis-master 6379:6379 &
    redis-cli -h 127.0.0.1 -p 6379 -a $REDIS_PASSWORD
```

## Bugs

1. pod has unbound immediate PersistentVolumeClaims: [follow link](https://medium.com/@thanawitsupinnapong/setting-up-redis-in-kubernetes-with-helm-and-manual-persistent-volume-f1d52fa1919f)

**Create Persistent Volume**
File: `redis-pv.yaml`

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis-data-redis-master-0
spec:
  storageClassName: manual
  capacity:
    storage: 8Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data/redis-master-0
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis-data-redis-slave-0
spec:
  storageClassName: manual
  capacity:
    storage: 8Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data/redis-slave-0
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis-data-redis-slave-1
spec:
  storageClassName: manual
  capacity:
    storage: 8Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data/redis-slave-1
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis-data-redis-slave-2
spec:
  storageClassName: manual
  capacity:
    storage: 8Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data/redis-slave-2
```

Apply:

```sh
kubectl apply -f redis-pv.yaml
```

**Install redis**

```sh
helm install redis bitnami/redis \
  --set master.persistence.enabled=true \
  --set master.persistence.path=/mnt/data/redis-master-0 \
  --set master.persistence.size=8Gi \
  --set master.persistence.storageClass=manual \
  --set slave.persistence.enabled=false \
  --namespace={namespace}
```
