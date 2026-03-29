

# FEATURES

Reverse Proxy
API Gateway and API Management
Load Balancing
Certificate Management
Kubernetes Ingress


# TRAEFIK PROVIDERS

A provider links an infrastructure component (orchestrator, container enginer, kv store, file) to Traefik

Some providers are
- docker                    -       i think the traefik container reads the labels on other containers and configures itself accordingly
- kubernetes                -       the traefik pod is an ingress controller, it reads the .spec.ingressClassName on an ingress and dynamically configures routes and services internally
- etcd
- mesos
- file


# COMPONENTS OF TRAEFIK

## Statically configured

    EntryPoints
                                 
    Provider

## Dynamically Configured

    Routers

    Services

    Middlewares

    Certificates (stores and options)


## Entrypoints

Entrypoints are points at which traffic enters traefik

Entrypoints can
- redirect connection from http to https
- forward header configuration
- override default TLS with user-defined TLS

In k8s this is the container ports that are being exposed
- 8000 - http
- 8443 - https
- 8080 - dashboard i think
- 9100 - metrics

NOTE: in our k8s we use cli arguments to configure entrypoints


## Routers

analyse requests by looking at host, path, headers, SSL, etc.
the job of the router is to do rule matching


## Services

a service can be assigned to one or more routers
it forwards the requests to backend servers
each service has its own loadbalancer (internal loadbalancer to traefik i think)
the target of the loadbalancer is a server
loadbalancers can be configured with health checks


## Middlewares

may update the request OR make decisions based on the request (authentication, rate limit, headers, etc.)


# CONFIGURAION

config file                     -    YAML or TOML
OR 
command line arguments 
OR 
environment variables



# TRAFFIC FLOW WITHIN TRAEFIK


client
  |
  |
  |
  V
Entrypoint             -        in k8s this is the traefik service in the traefik namespace
  |
  |
  |
  V
Router                 -        in k8s this would be configured because of the ingress resource
  |
  |
  |
  V
Service
  |
  |
  |
  V
backend server          -       



# HTTPS and TLS

## Default Certificate
If no cert is provided Traefik generates and uses a self signed cert

## User Defined
A user provides certificates directly to Traefik which will then be applied to matching entrypoints

## Automated
Traefik uses Let's Encrypt to generate certificates automatically on a per request basis

Let's Encrypt Challenge types
  HTTP Challenge      -       LE gives a token to Traefik which is then served back to LE for verification via http://<your_domain>/.well-known/acme-challenge/<TOKEN>

  DNS Challenge       -       Traefik uses your DNS providers API to place DNS TXT recoreds in your Domain records to verify ownership

  TLS Challenge       -       LE performs a handshake between Traefik and LE on port 443

  

# KUBERNETES

## CHARTS

The traefik-crds and traefik implementations are both via helm charts from the same repository (https://traefik.github.io/charts) and these helm charts do not follow the same numbering system. 
To determine the next candidate for upgrade you need to
 - Look at the version of the chart that has been deployed for traefik-crds. Then look for a later version of the chart in the repository (that you want to upgrade to)
 - Read the Chart.yaml from the later version and look for annotations.artifacthub.io/changes
   this will list the application version for Traefik, Traefik proxy. In specific look for the chore(release) line to see the chart for the Traefik application itself.


## CRDs

accesscontrolpolicies           hub.traefik.io/v1alpha1
apiaccesses                     hub.traefik.io/v1alpha1
apibundles                      hub.traefik.io/v1alpha1
apiplans                        hub.traefik.io/v1alpha1
apiportals                      hub.traefik.io/v1alpha1
apiratelimits                   hub.traefik.io/v1alpha1
apis                    		    hub.traefik.io/v1alpha1
apiversions                     hub.traefik.io/v1alpha1
ingressroutes                   traefik.io/v1alpha1
ingressroutetcps                traefik.io/v1alpha1
ingressrouteudps                traefik.io/v1alpha1
middlewares                     traefik.io/v1alpha1
middlewaretcps                  traefik.io/v1alpha1
serverstransports               traefik.io/v1alpha1
serverstransporttcps            traefik.io/v1alpha1
tlsoptions                      traefik.io/v1alpha1
tlsstores                       traefik.io/v1alpha1
traefikservices                 traefik.io/v1alpha1


As for the externalTrafficPolicy, that influences from which node an externalIp is exposed. 

When set to "Local", it is only exposed on the node for which the application backing the service is running. 

When set to "Cluster" (the default), it is exposed on all nodes. That's important for BGP. If your router doesn't support ECMP w/BGP, then you must only use Local. 
If they do support ECMP w/BGP, then multiple routes will be published for the same externalIP. You can see this here with "Local", there are some efficiency gains as it doesn't need to bounce around the cluster. But there's also the potential to saturate a single node, and failover could take longer or there may be downtime if you lose a switch (in the case with our setup, where we peer against 2 switches per site). There's a caveat with "Cluster" policy though - the backend application may never see the real client IP. That should be fine with Ingress though, as they'll set an X-Forwarded-For header and backend web server are able to see that and log the real IP anyway. For non-web apps; not much we can do & that's just the way it is.


---

Something of interest. Traefik's default behaviour for ingress is to bypass the backend service entirely and direct traffic straight to pods. That behaviour can be changed with the traefik.ingress.kubernetes.io/service.nativelb annotation on the backend service. When set to true, traefik will proxy traffic via that service.


And you can observe that behaviour with hubble. ie hubble observe --service http-helloworld -f
Default, you won't see anything hit that service. When set to true, you will.




Traefik reuses the established connections to the backends for performance purposes. This can prevent the requests load balancing between the replicas from behaving as one would expect when the option is set

except when you set that annotation to true, it changes that behaviour. connections no longer persist.

https://doc.traefik.io/traefik/reference/routing-configuration/kubernetes/ingress/#on-service