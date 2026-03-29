


# CRD's

cleanuppolicies.kyverno.io
clustercleanuppolicies.kyverno.io
clusterephemeralreports.reports.kyverno.io
clusterpolicies.kyverno.io
ephemeralreports.reports.kyverno.io
globalcontextentries.kyverno.io
policies.kyverno.io
policyexceptions.kyverno.io
updaterequests.kyverno.io


# API Resources

NAME                                SHORTNAMES                          APIVERSION                                NAMESPACED   KIND
cleanuppolicies                     cleanpol                            kyverno.io/v2                             true         CleanupPolicy
clustercleanuppolicies              ccleanpol                           kyverno.io/v2                             false        ClusterCleanupPolicy
clusterpolicies                     cpol                                kyverno.io/v1                             false        ClusterPolicy
globalcontextentries                gctxentry                           kyverno.io/v2alpha1                       false        GlobalContextEntry
policies                            pol                                 kyverno.io/v1                             true         Policy
policyexceptions                    polex                               kyverno.io/v2                             true         PolicyException
updaterequests                      ur                                  kyverno.io/v2                             true         UpdateRequest
clusterephemeralreports             cephr                               reports.kyverno.io/v1                     false        ClusterEphemeralReport
ephemeralreports                    ephr                                reports.kyverno.io/v1                     true         EphemeralReport


# VALIDATING AND MUTATING WEBHOOKS

```shell
$ k get validatingwebhookconfiguration
NAME                                            WEBHOOKS   AGE

kyverno-cleanup-validating-webhook-cfg          1          128d
kyverno-exception-validating-webhook-cfg        1          128d
kyverno-global-context-validating-webhook-cfg   1          128d
kyverno-policy-validating-webhook-cfg           1          128d
kyverno-resource-validating-webhook-cfg         1          128d
kyverno-ttl-validating-webhook-cfg              1          128d


$ k get mutatingwebhookconfiguration
NAME                                    WEBHOOKS   AGE

kyverno-policy-mutating-webhook-cfg     1          128d
kyverno-resource-mutating-webhook-cfg   0          128d
kyverno-verify-mutating-webhook-cfg     1          128d
```




# COMPONENTS

deployment.apps/kyverno-admission-controller
deployment.apps/kyverno-background-controller
deployment.apps/kyverno-cleanup-controller
deployment.apps/kyverno-reports-controller



## Troubleshooting

https://kyverno.io/docs/troubleshooting/
