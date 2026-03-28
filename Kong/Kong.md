
SERVICES
/services

host: can be a url or an upstream

Routes
/services/\<service\>/routes

Plugins

Plugins
/services/\<service\>/plugins

Consumers



ROUTES - rules that match requests to associated services, one route can
reference multiple end points
/routes

Plugins
/routes/\<route\>/plugins




UPSTREAMS or LOAD BALANCER - an upstream represents a virtual hostname
and can be used to health check, circuit break, and load balance
incoming requests over
multiple [target](https://docs.konghq.com/gateway/latest/admin-api/#target-object) backend
services
/upstreams

Targets
/upstreams/\<upstream\>/targets




CONSUMERS
/consumers

Plugins
/consumers/\<consumer\>/plugins

Groups

Credentials
Basic
API KEYS
HMAC
OAUTH2
JWT



PLUGINS
/plugins





CREDENTIALS






CONFIGURATION MANAGEMENT OF KONG

admin api
decK
Kong Kong Konnect





SOME COMMON PLUGINS

rate-limit

proxy-cache
/proxy-cache

key-auth
