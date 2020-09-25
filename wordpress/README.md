# Deploy wordpress

> Make sure that you have:
>
> - Helm v3+ [installed](https://helm.sh/docs/intro/install/)

Create persistent volume:

File: `persistent-volume.yaml`

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: data-wordpress
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /mnt/data
```

- Get and edit values.yaml file

```none
$ curl -fsSL -o values-production.yaml https://raw.githubusercontent.com/bitnami/charts/master/bitnami/wordpress/values-production.yaml
```

Should be changed:

> - wordpressUsername
> - wordpressPassword
> - service.type: `ClusterIP` for use traefik
> - allowEmptyPassword: false
> - ingress.enabled
> - ingress.hostname
> - ingress.annotations
> - ingress.tls
> - extraTls: uncomment if ingress.tls is true
> - persistence.storageClass: set value from `persistent-volume.yaml`
> - securityContext:
>   > enabled: true
>   > fsGroup: 0
>   > runAsUser: 0
> - MariaDB chart configuration or External Database Configuration

Install use the command:

```none
helm upgrade --install wordpress -f values-production.yaml bitnami/wordpress
```

**[Wordpress references files](../values-file/wordpress)**
