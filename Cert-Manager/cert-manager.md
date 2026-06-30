


# COMPONENTS

cert-manager-controller

cainjector

acmesolver

webhook

startupapicheck



# CONCEPTS

## CLUSTERISSUER

can read an annotation on ingress and then,
generates a private key,
then creates a CSR using the private key,
gets a certificate issued and
puts the certificate (tls.crt) and private key (tls.key) into a secret resource as named in the ingress tls section





# K8S RESOURCES

```k api-resources | grep cert-manager```

challenges                                                              acme.cert-manager.io/v1                   true         Challenge
orders                                                                  acme.cert-manager.io/v1                   true         Order
certificaterequests                 cr,crs                              cert-manager.io/v1                        true         CertificateRequest
certificates                        cert,certs                          cert-manager.io/v1                        true         Certificate
clusterissuers                      ciss                                cert-manager.io/v1                        false        ClusterIssuer
issuers                             iss                                 cert-manager.io/v1                        true         Issuer








# COMMANDS

```shell

# Find the status of a certificate
cmctl status certificate hubble-ui-tls


```