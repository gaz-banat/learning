
# CLUSTERSECRETSTORE

```bash
k get clustersecretstores.external-secrets.io vault-backend-global -o yaml | ditch_meta.sh
```

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ClusterSecretStore
metadata:
  annotations:
    meta.helm.sh/release-name: external-secrets
    meta.helm.sh/release-namespace: external-secrets
  labels:
    app.kubernetes.io/instance: external-secrets
    app.kubernetes.io/managed-by: Helm
    helm.toolkit.fluxcd.io/name: external-secrets
    helm.toolkit.fluxcd.io/namespace: flux-system
  name: vault-backend-global
spec:
  provider:
    vault:
      auth:
        jwt:
          kubernetesServiceAccountToken:
            serviceAccountRef:
              audiences:
                - vault://
              name: external-secrets
              namespace: external-secrets
          path: jwt-opsk8s-dev
          role: eso
      caProvider:
        key: DigiCertGlobalG2CA1
        name: trusted-ca-bundle
        namespace: default
        type: ConfigMap
      path: secret                                      # the path secret/ is already specified, so dont add in external secret
      server: https://vault.aarnet.net.au
      version: v2
  retrySettings:
    maxRetries: 3
    retryInterval: 10s
```


# EXTERNAL SECRET 

```bash
k get es aarnet-repo-creds -o yaml -n argocd | ditch_meta.sh
```

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  annotations: {}
  labels:
    argocd.argoproj.io/secret-type: repo-creds
  name: aarnet-repo-creds
  namespace: argocd
spec:
  refreshInterval: 60m
  secretStoreRef:
    kind: ClusterSecretStore
    name: vault-backend-global
  data:
    - secretKey: username                     # 'username' will become the key in .data section of the new secret 'aarnet-repo-creds'
      remoteRef:
        conversionStrategy: Default
        decodingStrategy: None
        key: k8s/dev/aarnet-repo              # this is the path in vault with the kv pairs - vault kv get secret/k8s/dev/aarnet-repo
        metadataPolicy: None
        property: username                    # 'username' is the key in the above path
    - secretKey: password
      remoteRef:
        conversionStrategy: Default
        decodingStrategy: None
        key: k8s/dev/aarnet-repo
        metadataPolicy: None
        property: password
    - secretKey: type                               
      remoteRef:
        key: k8s/dev/aarnet-repo                          
        property: type
        conversionStrategy: Default
        decodingStrategy: None
        metadataPolicy: None
    - secretKey: url
      remoteRef:
        conversionStrategy: Default
        decodingStrategy: None
        key: k8s/dev/aarnet-repo
        metadataPolicy: None
        property: url

  target:
    creationPolicy: Owner
    deletionPolicy: Retain
```


----

# Resync secret

```shell
secret=<secret>
namespace=<namespace>
kubectl annotate es $secret force-sync=$(date +%s) --overwrite -n $namespace
```

# Templated external secrets

Look at this example - https://external-secrets.io/latest/guides/templating/