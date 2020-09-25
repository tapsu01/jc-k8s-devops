# Deploy MySQL And PhpMyAdmin

> Make sure that you have:
>
> - Helm v3+ [installed](https://helm.sh/docs/intro/install/)

## Create PersistentVolume

File: `persistent-volume.yaml`

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: data-mysql-master-0
spec:
  storageClassName: manual
  capacity:
    storage: 8Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data/mysql-master-0
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: data-mysql-slave-0
spec:
  storageClassName: manual
  capacity:
    storage: 8Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data/mysql-slave-0
```

> Note: initialize persistent volume quantity by master + slave quantity

## Install MySQL

Get and change config value

```none
$ curl -fsSL -o values-production.yaml https://raw.githubusercontent.com/bitnami/charts/master/bitnami/mysql/values-production.yaml
```

> Should be changed:
>
> - root.forcePassword: false
> - root.injectSecretsAsVolume: false
> - db.user
> - db.password
> - db.name
> - db.forcePassword: true
> - db.injectSecretsAsVolume: false
> - uncomment persistence.storageClass, set to storageClassName from `persistent-volume.yaml`
> - slave.replicas
> - livenessProbe.enabled: false
> - readinessProbe.enabled: false
> - service:
>   > - use Traefik: change type to `ClusterIP`

Intall MySQL with values file

```none
$ helm install mysql -f values-production.yaml bitnami/mysql
```

Watch the deployment status using the command

```none
kubectl get pods -w --namespace default
```

Services:

```none
echo Master: mysql.default.svc.cluster.local:3306
echo Slave:  mysql-slave.default.svc.cluster.local:3306
```

Administrator credentials:

```none
echo Username: root
echo Password : $(kubectl get secret --namespace default mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode)
```

To connect to your database:

1. Run a pod that you can use as a client:

```none
kubectl run mysql-client --rm --tty -i --restart='Never' --image  docker.io/bitnami/mysql:8.0.21-debian-10-r46 --namespace default --command -- bash
```

2. To connect to master service (read/write):

```none
mysql -h mysql.default.svc.cluster.local -uroot -p jetcare_wordpress
```

3. To connect to slave service (read-only):

```none
mysql -h mysql-slave.default.svc.cluster.local -uroot -p jetcare_wordpress
```

**[MySQL references files](../values-file/mysql)**

## Install phpmyadmin

Get and change config value

```none
$ curl -fsSL -o values.yaml https://raw.githubusercontent.com/bitnami/charts/master/bitnami/phpmyadmin/values.yaml
```

Should be changed:

> - ingress.enabled
> - ingress.annotations
> - ingress.hosts.name
> - ingress.hosts.tls
> - ingress.hosts.tlsSecret

Install use the command:

```none
helm upgrade --install phpmyadmin bitnami/phpmyadmin -f values.yaml
```

Response:

```none
NAME: phpmyadmin
LAST DEPLOYED: Fri Sep 25 00:37:20 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
1. Get the application URL by running these commands:
  You should be able to access your new phpMyAdmin installation through
  http://your-cluster-ip

  Find out your cluster ip address by running:
  $ kubectl cluster-info

2. How to log in
phpMyAdmin has not been configure to point to a specific database. Please provide the db host,
username and password at log in or upgrade the release with a specific database:

$ helm upgrade phpmyadmin bitnami/phpmyadmin --set db.host=mydb

** Please be patient while the chart is being deployed **
```

**Exposing the phpmyadmin dashboard**

```none
kubectl port-forward --namespace default svc/phpmyadmin 8080:80
```

Accessible with the url: http://127.0.0.1:80/

**[PhpMyAdmin references files](../values-file/phpmyadmin)**
