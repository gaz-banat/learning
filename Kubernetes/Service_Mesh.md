

# WHAT IS A SERVICE MESH

Ingress
    Load Balancing (N-S)
    Protocol parsing
    Path based routing
L7 Traffic Management
    Service Load Balancing (E-W)
    Rules (canary rollouts, retries etc)
Identity based security
Observability


# SERVICE MESH FUNCTIONALITY

## L7 - Userspace based features
Request load balancing
Retry
Timeout
Circuit breaking
JWT validation
header manipulation
traffic splitting
mirroring
locality aware routing
request fault injection
request level authorization
JWT claims
headers
OIDC
ExtAuth
Rate Limit

## L7 - Userspace/Kernelspace
Observing request count
http 5xx
request latency
request size

## L4 - Userspace && Kernelspace
mTLS mutual auth
protocol/port authorization

## L4 - Userspace/Kernelspace
Connection metrics
connection load balancing

## L4/L3 - Kernelspace
Network policy





# ISSUES WITH SIDE-CAR PROXIES

container race conditions
security - cert/key material
Difficult to size, easy to overprovision
apps need to be aware
upgrades can be challenging
Jobs/Cronjobs have issues ??
Apps need to be aware ??



# SIDECAR-LESS Service Mesh

Fully transparent cannot opt out


