


TLSRoute        -       TLS Passthrough



# Gateway API - Custom Resources

```shell
$ k api-resources --api-group gateway.networking.k8s.io
NAME                 SHORTNAMES   APIVERSION                     NAMESPACED   KIND
backendtlspolicies   btlspolicy   gateway.networking.k8s.io/v1   true         BackendTLSPolicy
gatewayclasses       gc           gateway.networking.k8s.io/v1   false        GatewayClass
gateways             gtw          gateway.networking.k8s.io/v1   true         Gateway
grpcroutes                        gateway.networking.k8s.io/v1   true         GRPCRoute
httproutes                        gateway.networking.k8s.io/v1   true         HTTPRoute
listenersets         lset         gateway.networking.k8s.io/v1   true         ListenerSet
referencegrants      refgrant     gateway.networking.k8s.io/v1   true         ReferenceGrant
tcproutes                         gateway.networking.k8s.io/v1   true         TCPRoute
tlsroutes                         gateway.networking.k8s.io/v1   true         TLSRoute
udproutes                         gateway.networking.k8s.io/v1   true         UDPRoute
```