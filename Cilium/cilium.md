

# LINKS

Cilium Repo					-		https://github.com/cilium/cilium
Releases for Cilium			-		https://github.com/cilium/cilium/releases	
Specific release			-		https://github.com/cilium/cilium/releases/tag/v1.17.6

Helm Repo					-		https://helm.cilium.io
Chart versions				-		helm repo update && helm search repo cilium/cilium --versions
Reference for Helm			-		https://docs.cilium.io/en/stable/helm-reference/ AND https://artifacthub.io/packages/helm/cilium/cilium

AARNet Helm Repo			-		oci://aplregistry.aarnet.edu.au/helm-charts/helm.cilium.io/cilium


# CILIUM PROJECTS

CNI
Service Mesh
Hubble
Tetragon					-		Runtime Security Enforcement
									Runtime Security Observability (Syscall, File, Privilege, Network, etc)



# CILIUM CAPABILITIES

## Networking
Transparent Encryption - Can use IPSec or WireGuard
Load Balancing - Cilium can replace kube-proxy and be used as a stand alone load balancer
Cilium and Hubble export metrics

## Network Security 
This is Identity Aware Network Policy Enforcement
Cilium assigns an identity to groups of application containers based on relevant metadata, such as Kubernetes labels. 
The identity is then associated with all network packets emitted by the application containers

## Network Observability
This is using hubble

## Service Mesh

## Cluster Mesh (Multi-cluster Networking)
Cilium can do a cluster mesh

## Runtime Security
Security enforced at runtime

## Runtime Observability




# COMPONENTS

## Cilium components:

Operator									-	responsible for duties that need to be handled once for the entire cluster, operator is not in critical path

Agent (Daemon, Per Node)					-	runs as daemonset, the main cilium program
												interacts with kube-apiserver to synchronize cluster state
												interacts with linux kernel to load eBPF programs and update eBPF maps
												interacts with cilium cni plugin executable, via a filesystem socket
												creates on demand DNS and Envoy proxies as needed based on requested network policy
												creates Hubble gRPC services when hubble is enabled
												A REST API (the Server part)
												In-Agent cli called ‘cilium’ (the Client part)

CNI Plugin ('cilium-cni')					-	/opt/cni/bin/cilium-cni, the kubelet calls this binary and this binary calls the cilium agent pod
												/etc/cni/net.d/05-cilium.conflist is the configuration

CLI tool outside of Kubernetes (‘cilium’)	-   don’t confuse this with the in-agent cli with the same name

Relay called ‘hubble-relay’					-	hubble observer service, hubble peer service

GUI called ‘hubble-ui’						-	has frontend and backend as separate containers

Client (CLI) called ‘hubble’

Cluster Mesh API Server						-	optional component deployed when cluster mesh feature is enabled


Envoy										-	The Envoy Proxy is an open source, high-performance, small-footprint edge and service proxy. 
												It works similarly to software load balancers like NGINX and HAProxy.
												used when something can't be done in eBPF

## eBPF

	maps										-	Connection Tracking, NAT, Neighbor Table, Endpoints, IP cache, Load Balancer, Policy, Proxy Map


## Data Store components:

	Kubernetes CRD’s

	Key-Value store (etcd supported)





# CONCEPTS

Cilium Endpoints		-	all application containers that share a common IP address are grouped into an Endpoint (so a pod)

Cilium Identity			-	an identity is a number + kv labels assigned to an endpoint
							many endpoints can share an identity (like say the n pods of a deployment)
							an identity is unique cluster wide, 
							The unique numeric identifier associated with each identity is then used by eBPF programs in very fast lookup in the network datapath
							network policy is applied based on identity



## NETWORKING Concepts

Routing 
	
	Routing Modes
		Tunnel/Encapsulation
			VXLan
			Geneve
		Native
		AWS ENI
		Google Cloud


IPAM

	IPAM Features
		Tunnel Routing
		Direct Routing
		CIDR Configuration
		Multiple CIDRs per cluster
		Multiple CIDRs per node
		Dynamic CIDR/IP allocation

	IPAM Modes (check the matrix for what mode can provide what feature)

		Cluster Scope (default)
		Kubernetes Host Scope mode (ipam.mode=kubernetes)
		Multi-Pool
		Azure IPAM
		Azure Delegated IPAM
		AWS ENI
		GKE
		CRD-Backed
	

Masquerading

	eBPF based (bpf.masquerade=true)
	iptables based


IPV4 fragment handling


Encryption

	WireGuard		-	in-kernel, peer-based, vpn made by exchanging public keys, 
	IPSec



## LOAD BALANCING Concepts


Types:
	north-south
	east-west



## NETWORKPOLICY Concepts

	NetworkPolicy							-		namespaced

	CiliumNetworkPolicy						-		namespaced, i think this controls policy for pods

	CiliumClusterwideNetworkPolicy			-		clusterwide, i think this controls policy for nodes rather than for pods 

NOTE: https://networkpolicy.io


### Working with Cilium Network Policies (namespaced and clustered)


NOTE: https://editor.networkpolicy.io/


There are 3 parts to a policy

select what the policy applies to
	nodeSelector								-	select nodes in a cluster
	endpointSelector							-	select pods in the cluster							

specify the ingress part
	- fromEntities
	  - world									-	from outside the cluster
	  - cluster									-	from anywhere in the cluster (endpoint or node?)
	- fromEndpoints								-	from one or more pods in a namespace
	- fromNodes									-	from one or more nodes in the cluster

specify the egress part
	- toEntities
		- world									-	to outside the cluster
		- cluster								-	to anywhere in the cluster	
	- toEndpoints								-	to one or more pods in a namespace
	- toFQDNs									-	to a fully qualified domain name


After you specify what an ingress or egress applies to you can specify port+protocol for that rule as well, you can specify matching labels for the endpoint as well

e.g.
- toEndpoints:
  - matchLabels:
	  app: web1
  toPorts:
  - ports:
    - port: "53"
	  protocol UDP

- fromEntities:
  - cluster
  toPorts:
  - ports:
    - port: "8000"
	  protocol TCP


1. Selecting endpoints and nodes

if you declare an empty selector then cilium selects all

nodeSelector: {}      	# this means select all nodes in the cluster that the policy applies to
endpointSelector: {}	# this selects all endpoints (pods) in the namespace that the policy applies to

2. You can select all for a given entity, endpoint, node by using {}
- toEndpoints:
  - {}							-	this matches to all endpoints in the cluster


3. The moment you define an ingress or egress section a default deny at the end of that section will kick in

Look at this - egress and ingress have been defined without specifying rules, so the default deny at the end is stopping all traffic, basically no allow rules!

egress: {}		# deny all egress traffic to what has been specified by the selector
ingress: {}		# deny all ingress traffic to what has been specified by the selector


4. You can work in reverse and specify what is denied BUT THERE IS A CATCH HERE.. it seems that there is already a default deny for whatever is selected per cilium behaviour. 
So after the egress deny rules there is a default deny!!

egressDeny: []			# deny all egress
no ports in a rule		# apply to all ports




## BANDWIDTH MANAGEMENT Concepts



# MONITORING AND TROUBLESHOOTING

cilium operator metrics start with 'cilium_operator_'

cilium agent metrics
	Cluster Health 			- Statistics on unreachable nodes and agent health endpoints
	Node Connectivity 		- Statistics covering latency to nodes across the network
	Cluster Mesh 			- Statistics concerning peer clusters
	Datapath 				- Statistics related to garbage collection of connection tracking
	IPSec 					- Statistics associated with IPSec errors
	eBPF 					- Statistics on eBPF map operations and memory use
	Drops/Forwards (L3/L4) 	- Statistics on packet drops/forwards.
	Policy 					- Statistics on active policy
	Policy L7 (HTTP/Kafka) 	- Statistics for L7 policy redirects to embedded HTTP proxy
	Identity 				- Statistics concerning Identity to IP address mapping
	Kubernetes 				- Statistics concerning received Kubernetes events
	IPAM 					- IP address allocation statistics

hubble metrics
	DNS						- Statistics about DNS requests made
	Drop					- Statistics about packet drops
	Flow					- Statistics concerning total flows processed
	HTTP					- Statistics concerning HTTP requests
	TCP						- Statistics concerning TCP packets
	ICMP					- Statistics concerning ICMP packets
	Port DistributionStatistics concerning destination ports
NOTE: hubble metrics are about data flows rather than about hubbles own performance



# KUBERNETES RESOURCES

deployment/Cilium-Operator
ds/cilium
ds/cilium-envoy

deployment/hubble-ui
deployment/hubble-relay

cm/cilium-config

sa/
clusterrole/
clusterrolebinding/



# CRD’s

ciliumnodes										-		represents a node in cilium
ciliumnodeconfigs

ciliumendpoints
ciliumidentities

ciliumcidrgroups

ciliumendpointslices 							-		WE ARE NOT USING THIS

ciliumpodippools

ciliumloadbalancerippool

ciliumexternalworkloads


## Network Policies
ciliumnetworkpolicies							-		namespace scoped - for traffic in and out of a namespace
ciliumclusterwidenetworkpolicies				-		cluster scoped - for traffic in and out of the nodes
ciliuml2announcementpolicies


## BGP control plane
ciliumbgpclusterconfig							-		Defines BGP instances and peer configurations that are applied to multiple nodes
ciliumbgppeerconfig								-		A common set of BGP peering setting. It can be used across multiple peers
ciliumbgpadvertisement							-		Defines prefixes that are injected into the BGP routing table
ciliumbgpnodeconfigoverrides					-		Defines node-specific BGP configuration to provide a finer control.

ciliumbgpnodeconfigs


### ignore this one
ciliumbgppeeringpolicies						-		this will be deprecated in the future (replaced by ciliumbgpclusterconfig)





# COMMANDS

General:

```shell
cilium version

cilium status [--wait]

cilium connectivity test --request-timeout 20s --connect-timeout 10s

# What is the kubernetes context for cilium
cilium context

# Get the cilium config
cilium config view
or
kubectl get cm cilium-config -o yaml | yq -y '.data'

# Get the features active on a node
cilium features status --node <node>
```


Endpoints and Identities:

```shell
# Looking at endpoints
kubectl get ciliumendpoint -A -o wide

# Get identities
kubectl get ciliumidentities -A
```

Debugging:

```shell
kubectl -n <cilium_cni_namespace> exec ds/cilium --	cilium-dbg status [ --verbose ]
kubectl -n <cilium_cni_namespace> exec ds/cilium --	ip link
kubectl -n <cilium_cni_namespace> exec ds/cilium --	cilium-dbg endpoint list
kubectl -n <cilium_cni_namespace> exec ds/cilium -- cilium-dbg service list
kubectl -n <cilium_cni_namespace> exec ds/cilium --	cilium-dbg policy get 
kubectl -n <cilium_cni_namespace> exec ds/cilium --	cilium-dbg monitor
```


BGP:

```shell
$ cilium bgp peers
Node                        Local AS     Peer AS      Peer Address   Session State   Uptime      Family         Received   Advertised
nsw-artm-opsk8stest-iwrk1   4200020003   4200012433   10.253.172.2   established     50h3m58s    ipv4/unicast   0          2    
                            4200020003   4200012434   10.253.172.3   established     50h3m58s    ipv4/unicast   0          2    
nsw-etca-opsk8stest-iwrk1   4200020003   4200012335   10.253.168.2   established     37h54m21s   ipv4/unicast   0          2    
                            4200020003   4200012336   10.253.168.3   established     37h54m19s   ipv4/unicast   0          2    
nsw-mcqm-opsk8stest-iwrk1   4200020003   4200012529   10.253.176.2   established     50h3m48s    ipv4/unicast   0          2    
                            4200020003   4200012530   10.253.176.3   established     50h3m45s    ipv4/unicast   0          2


$ cilium bgp routes advertised ipv4 unicast 
Node                       VRouter      Peer           Prefix              NextHop         Age         Attrs
nsw-artm-opsk8s-iwrkdev1   4200020003   10.253.172.2   202.158.207.18/32   10.253.173.22   69h42m51s   [{Origin: i} {AsPath: 4200020003} {Nexthop: 10.253.173.22}]   
                           4200020003   10.253.172.3   202.158.207.18/32   10.253.173.22   69h42m51s   [{Origin: i} {AsPath: 4200020003} {Nexthop: 10.253.173.22}]   
nsw-etca-opsk8s-iwrkdev1   4200020003   10.253.168.2   202.158.207.18/32   10.253.169.22   69h32m19s   [{Origin: i} {AsPath: 4200020003} {Nexthop: 10.253.169.22}]   
                           4200020003   10.253.168.3   202.158.207.18/32   10.253.169.22   69h32m19s   [{Origin: i} {AsPath: 4200020003} {Nexthop: 10.253.169.22}]   
nsw-mcqm-opsk8s-iwrkdev1   4200020003   10.253.176.2   202.158.207.18/32   10.253.177.22   69h16m28s   [{Origin: i} {AsPath: 4200020003} {Nexthop: 10.253.177.22}]   
                           4200020003   10.253.176.3   202.158.207.18/32   10.253.177.22   69h16m28s   [{Origin: i} {AsPath: 4200020003} {Nexthop: 10.253.177.22}]   
```


Hubble observe:

```shell
cilium [-n cilium-cni] hubble port-forward&
hubble status
hubble observe -n <namespace> --pod <pod> --protocol <protocol> --since [1m|3s] -f --verdict DROPPED | FORWARDED
OR
hubble observe --node-name nsw-artm-opsk8s-ctrldev1 --traffic-direction ingress --type trace:to-stack --verdict DROPPED -f
fg 1
ctrl-c
```


Updating the config:

```shell
helm upgrade

cilium config set
```


Versions:

```shell

$ k describe -n kube-system deployment cilium-operator | grep -i image
    Image:      aplregistry.aarnet.edu.au/quay.io/cilium/operator-generic:v1.17.6@sha256:91ac3bf7be7bed30e90218f219d4f3062a63377689ee7246062fa0cc3839d096


$ k describe -n kube-system ds cilium | grep -i image
    Image:      aplregistry.aarnet.edu.au/quay.io/cilium/cilium:v1.17.6@sha256:544de3d4fed7acba72758413812780a4972d47c39035f2a06d6145d8644a3353
    Image:      aplregistry.aarnet.edu.au/quay.io/cilium/cilium:v1.17.6@sha256:544de3d4fed7acba72758413812780a4972d47c39035f2a06d6145d8644a3353
    Image:      aplregistry.aarnet.edu.au/quay.io/cilium/cilium:v1.17.6@sha256:544de3d4fed7acba72758413812780a4972d47c39035f2a06d6145d8644a3353
    Image:      aplregistry.aarnet.edu.au/quay.io/cilium/cilium:v1.17.6@sha256:544de3d4fed7acba72758413812780a4972d47c39035f2a06d6145d8644a3353
    Image:      aplregistry.aarnet.edu.au/quay.io/cilium/cilium:v1.17.6@sha256:544de3d4fed7acba72758413812780a4972d47c39035f2a06d6145d8644a3353
    Image:       aplregistry.aarnet.edu.au/quay.io/cilium/cilium:v1.17.6@sha256:544de3d4fed7acba72758413812780a4972d47c39035f2a06d6145d8644a3353


$ k describe -n kube-system ds cilium-envoy | grep -i image
    Image:      aplregistry.aarnet.edu.au/quay.io/cilium/cilium-envoy:v1.33.4-1752151664-7c2edb0b44cf95f326d628b837fcdd845102ba68@sha256:318eff387835ca2717baab42a84f35a83a5f9e7d519253df87269f80b9ff0171


$ k describe -n kube-system deployment hubble-relay | grep -i image
    Image:      aplregistry.aarnet.edu.au/quay.io/cilium/hubble-relay:v1.17.6@sha256:7d17ec10b3d37341c18ca56165b2f29a715cb8ee81311fd07088d8bf68c01e60


$ k describe -n kube-system deployment hubble-ui | grep -i image
    Image:        aplregistry.aarnet.edu.au/quay.io/cilium/hubble-ui:v0.13.2@sha256:9e37c1296b802830834cc87342a9182ccbb71ffebb711971e849221bd9d59392
    Image:      aplregistry.aarnet.edu.au/quay.io/cilium/hubble-ui-backend:v0.13.2@sha256:a034b7e98e6ea796ed26df8f4e71f83fc16465a19d166eff67a03b822c0bfa15

$ k exec -n kube-system ds/cilium -- cilium version
Defaulted container "cilium-agent" out of: cilium-agent, config (init), apply-sysctl-overwrites (init), mount-bpf-fs (init), clean-cilium-state (init), install-cni-binaries (init)
Client: 1.17.6 ee3ca4a7 2025-07-14T15:28:45+00:00 go version go1.24.5 linux/amd64
Daemon: 1.17.6 ee3ca4a7 2025-07-14T15:28:45+00:00 go version go1.24.5 linux/amd64

```