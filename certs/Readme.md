# Cert-manager

[References](https://cert-manager.io/docs/installation/kubernetes/)

- Create cert-manager namespace

```none
kubectl create ns cert-manager
```

**Option 1: Installing with regular manifests (recommend)**

```none
# Kubernetes 1.16+
$ kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.0.1/cert-manager.yaml

# Kubernetes <1.16
$ kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.0.1/cert-manager-legacy.yaml
```

**Output should be like below**

```none
NAME: cert-manager
LAST DEPLOYED: Tue Sep 22 10:18:36 2020
NAMESPACE: cert-manager
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
cert-manager has been deployed successfully!

In order to begin issuing certificates, you will need to set up a ClusterIssuer
or Issuer resource (for example, by creating a 'letsencrypt-staging' issuer).

More information on the different types of issuers and how to configure them
can be found in our documentation:

https://cert-manager.io/docs/configuration/

For information on how to configure cert-manager to automatically provision
Certificates for Ingress resources, take a look at the `ingress-shim`
documentation:

https://cert-manager.io/docs/usage/ingress/
```

**Create `issuer`**

File: `letsencrypt-issuer.yaml`

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    # You must replace this email address with your own.
    # Let's Encrypt will use this to contact you about expiring
    # certificates, and issues related to your account.
    email: user@example.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      # Secret resource that will be used to store the account's private key.
      name: example-issuer-account-key
    # Add a single challenge solver, HTTP01 using nginx
    solvers:
      - http01:
          ingress:
            class: nginx
```

```none
kubectl apply -f letsencrypt-issuer.yaml
```

**Note**

> `example-issuer-account-key` use for cert-manager.io/cluster-issuer of values.yaml helm charts or ingress controller
