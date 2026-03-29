
ServiceAccount/dex


Role/dex
Rolebinding/dex


Clusterrole/dex
Clusterrolebinding/dex-cluster

---

es/dex-clients                                              # secret/dex-clients

es/dex-ldap                                                 # secret/dex-ldap

es/aarnet-root-ca                                           # secret/aarnet-root-ca     -   so that dex can verify the ssl cert when talking to ldap

---

secret/dex-config-XXXXXXXX                                  # configuration for dex in a key called config.yaml
    issuer url
    logging
    storage
    tls cert and key
    connectors
        ldap connector config
    static clients
        argocd
        grafana
        prometheus
        alertmanager
        talos-poc-k8s
    oauth2


---

deployment/dex


service/dex
    5556 --> 80
    5554 --> 443
    5558 --> telemetry (5558)


ingress/dex                                                 # certificate/dex-tls is generated via annotation, followed by secret/dex-tls
    oidc-opsk8slab.aarnet.net.au                            # secret/dex-tls (tls.crt is subject=O=AARNet, CN=oidc-opsk8slab.aarnet.net.au, issuer=CN=AARNet Kubernetes Intermediate CA)


---

poddisruptionbudget/dex
hpa/dex


---

Clusterrolebinding/oidc:cluster-admin                       # binds clusterrole/cluster-admin to group/oidc:lxadmsu 



---
# DOCUMENTATION

so my understanding is:
- kauth authenticates with dex to get a token (this requires the client secret)
- kauth configures a kubectl user with the token
- kubectl sends the token to kube api-server when you run a command
- the api server validates the token with dex (this requires the token, and the client id)
	- if it's valid, it'll check clusterroles before allowing the command to run
	- if its not valid, you get an error


https://dexidp.io/docs/guides/kubernetes/

Kubernetes Authentication Through Dex
Overview This document covers setting up the Kubernetes OpenID Connect token authenticator plugin with dex. It also contains a worked example showing how the Dex server can be deployed within Kubernetes.
Token responses from OpenID Connect providers include a signed JWT called an ID Token. ID Tokens contain names, emails, unique identifiers, and in dex’s case, a set of groups that can be used to identify the user. OpenID Connect providers, like dex, publish public keys; the Kubernetes API server understands how to use these to verify ID Tokens.