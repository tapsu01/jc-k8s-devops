# Nats

```none
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install nats bitnami/nats --namespace={namespace} --set auth.password={password}
```

NOTES: Please be patient while the chart is being deployed

NATS can be accessed via port 4222 on the following DNS name from within your cluster:

nats-client.{namespace}.svc.cluster.local

> In helm chart set nats.TRANSPORTER_URL: nats://nats-client.{namespace}.svc.cluster.local:4222

To get the authentication credentials, run:

```none
export NATS_USER=$(kubectl get cm --namespace {namespace} nats -o jsonpath='{.data.*}' | grep -m 1 user | awk '{print $2}')
export NATS_PASS=$(kubectl get cm --namespace {namespace} nats -o jsonpath='{.data.*}' | grep -m 1 password | awk '{print $2}')
echo -e "Client credentials:\n\tUser: $NATS_USER\n\tPassword: $NATS_PASS"
```

**Encode base64 for set in helm chart**

```none
echo -n <value> | base64
```

To access the Monitoring svc from outside the cluster, follow the steps below:

1. Get the NATS monitoring URL by running:

```none
echo "Monitoring URL: http://127.0.0.1:8222"
kubectl port-forward --namespace {namespace} svc/nats-monitoring 8222:8222
```
