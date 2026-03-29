
# PART I - MAIN KUBERNETES COMPONENTS

Master																					
   Mandatory
	API Server				respond to API requests from an authenticated source										CONTROL PLANE COMPONENT	RUNS IN A CONTAINER	
	Controller Manager		manages controller processes																CONTROL PLANE COMPONENT	RUNS IN A CONTAINER	
	Scheduler				schedules pods on various nodes																CONTROL PLANE COMPONENT	RUNS IN A CONTAINER	
	etcd					stores configuration																		CONTROL PLANE COMPONENT	RUNS IN A CONTAINER	
	DNS server				responds to DNS queries													
	Kubelet					main k8s service/agent on a node, it registers the node with the apiserver, 				this is a daemon
	Kube-proxy				IPAM plus other duties																		As a daemonset OR linux host process	
	CRI Runtime
	CNI Provider
	Systemd
  Optional
	CSI Provisioner			storage-provisioner
	Cloud Controller Manager														CONTROL PLANE COMPONENT	RUNS IN A CONTAINER

Worker
  Mandatory
	Kubelet																			this is a daemon
	Kube-proxy																		As a daemonset OR linux host process	kube-proxy
	Container Runtime
	CNI Provider
	Systemd



## API SERVER

API server is available at the svc https://kubernets.default.svc.cluster.local/ (given that cluster.local is the dns domain)	

NOTE: look at the flags with which the api server is started


Admission Controller
an admission controller is a piece of code that intercepts requests to the api server prior to persistence of the object but after the request is authenticated and authorized
Admission controllers are code within the Kubernetes API server that check the data arriving in a request to modify a resource.

Types of admission controllers
- validating		-	will validate something on the incoming resource
- mutating			-	will change something on the incoming resource


NOTE: List of admission controllers - https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#what-does-each-admission-controller-do

Some common admission controllers
PodSecurity

The 3 admission controllers that let you expand functionality in the Kube API
ImagePolicyWebhook
MutatingAdmissionWebhook
ValidatingAdmissionWebhook


### Commands

```shell
# I want to see all the apiserver endpoints
kubectl get endpoints

# I want to see all the api’s available on the cluster
kubectl get apiservices
                                                                                                              

# I want to see all the resources that the API server is aware about
kubectl api-resources -o wide


# What are ALL the admission controllers in a Kubernetes cluster
https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers


# Which admission plugins is the apiserver running with  (DOUBTFUL ABOUT THIS CAUS NOT ALL PLUGINS ARE BEING LISTED I THINK)
Look at the command /usr/local/bin/kube-apiserver flag --enable-admission-plugins
e.g. talosctl processes -n nsw-artm-opsk8stest-ctrl1 | grep kube-apiserver | sed 's| --|\n\t--|g'

# Which binary does the kube-apiserver start with
In a kube api pod the binary is ‘kube-apiserver’, e.g. kubectl exec <kube-api-pod> -- kube-apiserver -h 
```



## KUBE CONTROLLER MANAGER

The KCM (kube-controller-manager) is a set of control loops that run various components, called controllers, on a node that is on a control plane.
It is a single binary, but operates multiple control loops and, thus, controllers.

What is a controller:
- a controller is a just an app running in a pod that connects to the Kubernetes API
- a controller is a Kubernetes resource which manages other resources 
- it works with Kubernetes API to watch the current state of the system
- it uses a label selector to identify the resource to be managed


Controllers:
Node controller
Route controller
Service controller			-	 i think this is the component that reads a request for say a new LoadBalancer service and makes API calls needed to create a load balancer in the cloud.
Volume controller




## SCHEDULER

This component watches for newly created pods and places (schedules) them onto nodes

Scheduling principles
filtering - find a set of feasible nodes for the new pod (consider resources on nodes, etc)
Scoring - give a score to the nodes and rank them. Place pod on highest ranking node


Scheduling Policies

Scheduling Profiles





## DNS SERVER

.cluster.local							-	the default suffix in a kubernetes installation
.svc.cluster.local						-	the domain/suffix in which services live by default
<service-name>.<namespace>.<suffix>		-	e.g. api.project1.svc.cluster.local is the service ‘api’ from the namespace ‘project1’





## ETCD





## KUBELET

main k8s service/agent on a node, it registers the node with the apiserver, monitors pods and reports their health information to the api server
The API server is continually made aware that the kubelet on a node is healthy via a heartbeat in Kubernetes 1.17+. (The heartbeat is maintained by looking at the kube-node-lease namespace of a running cluster.)
Master node kubelet and does not register for cluster work


/etc/kubernetes/manifests 		-	this direcotry contains the yaml files for the control plane pods




## KUBE-PROXY

deals with individual host subnetting, manages iptables, expose services, forwards requests to pods/containers			
provide ip addresses for pods
provide ip addresses for services
creates a NodePort service
monitors changes to service objects and their endpoints and translates them into actual networking rules inside a node (it does by talking to the kube-api server)

Can work in 3 modes - 
	userspace				-	uses standard networking tools to redirect traffic, runs in userspace, slow and not efficient 
	kernelspace				-	Windows OS only
	iptables				-	uses the kernel's iptables firewall, based on netfilter, runs in userspace
								in iptables mode, kube-proxy attaches rules to the “NAT pre-routing” hook to implement its NAT and load balancing functions
	IPVS					-	uses the kernel's IP Virtual Server, based on netfilter, implements transport layer load balancing, usually called Layer 4 LAN switching,
								it provides better scalability and performance for large clusters, supports more sophisticated load balancing algorithms than iptables (e.g. round robin, lease connection, destination source hashing)





## CONTAINER RUNTIME INTERFACE (CRI)

an API for a Kubernetes node to talk to a container runtime/engine






## CONTAINER NETWORK INTERFACE (CNI)

Understand these terms:
IPAM
Dual Stack
eBPF
VxLAN
Overlay


Kubernetes can use CNI plug-ins to orchestrate networking. 
Everytime a pod or service is initialized or removed the default CNI plugin is called with the default configuration.
The CNI plugin is responsible for creating a pseudo interface, will attach the relevant underlay network, setup the IP routes for the pod namespace, and so on.


### Commands

I want to see the cni plugins that have been installed
On a node -
ls /etc/cni/net.d 		- this contains the CNI json configuraion file
ls /opt/cni/bin 		- this contains the CNI binary





## CONTAINER STORAGE INTERFACE

NOTE: https://kubernetes-csi.github.io/docs/

This is a standard for creating custom components to work with data storage - the standard comprises of controller service, node service and identity service.

The standard has a specification at each version - THERE MUST BE SOME WAY TO SHOW THE CSI VERSION FOR A GIVEN KUBERNETES VERSION

### CSI Driver

A CSI driver (aka plugin/provisioner) is needed to be installed for implementing the services and access to a storage system. 

From what I understand a CSI driver registers itself with the Kubernetes cluster

A CSI driver (aka plugin/provisioner) must contain at least these 2 following components 
(each component is implemented as a pod with one or more containers, and the CSI driver code is in one of the containers)


#### Controller component 													(in ceph this is the csi-cephfsplugin-provisioner)
- manages persistent external volumes (creating, deleting, etc.)
- Implemented as a gRPC server
- The kubernetes primitive is a StatefulSet or Deployment

This component talks to the control plane

The controller component pod is made up of
- a csi driver container containg the 
	- controller service (ControllerGetCapabilities(), CreateVolume(), PublishVolume(), UnpublishVolume(), DeleteVolume())
	- identity Service	(GetPluginInfo(), GetPluginCapabilities())					
- one or more sidecar containers implementing
	- external-provisioner							watches for PersistentVolumeClaim objects
	- external-attacher								watches for VolumeAttachment objects
	- external-resizer (optional)					watches for PersistentVolumeClaim objects
	- external-snapshotter (optional)				watches for VolumeSnapshot objects
- and perhaps other containers providing additional functionality like
	- livenessprobe

Note: the csi driver container talks to sidecar container/s using a UNIX domain socket which is shared via an emptyDir volume


#### Node component	 														(in ceph this is the csi-cephfsplugin)		
- mounts/unmounts persistent external volumes to a kubernetes node (think attaching and detaching volumes to and from nodes)
- Implemented as a gRPC server
- The kubernetes primitive is a DeamonSet (a pod for each node)

this component talks to the kubelet on the node via a UNIX domain socket which is shared via HostPath volume on the node host
An additional UNIX domain socket is used by the node-driver-registrar to register the driver to the kubelet

The node component pod is made up of
- a csi driver container containing the 
	- node service (NodeGetCapabilities(), NodeStageVolume(), NodePublishVolume(), NodeUnstageVolume(), NodeUnpublishVolume())
	- identity service (GetPluginInfo(), GetPluginCapabilities())
- a sidecar container that serves as a node-driver registrar


### StorageClass

A StorageClass will use a csidriver by specifying the driver in its provisioner field.

The StorageClass calls the CSI driver with parameters specified in it's parameters field

The StorageClass dictates the volumeBindingMode and the reclaimPolicy for the storage


### The process of provisioning a persistent volume

1. the external-provisioner sidecar ----- watches for -----> a PVC ---- which references ---- a STORAGECLASS
2. the external-provisioner ---then calls (from provisioner field, with parameters from parameters field of STORAGECLASS) CreateVolume() ----> CSI Driver (controller component) --- which provisions a volume on ---> Storage Provider
2. Kubelet ---uses----> CSI Driver (node component) ---to mount volume onto node from----> Storage Provider  ----finally getting a-----> PV



### CSI External-Snapshotter 

NOTE: https://github.com/kubernetes-csi/external-snapshotter AND https://kubernetes-csi.github.io/docs/external-snapshotter.html


CRD's:

volumesnapshotclasses.snapshot.storage.k8s.io						-		a storage class for snapshots, it ties to a csi driver
volumesnapshots.snapshot.storage.k8s.io								-		a user submits this resource to the cluster - think of it as a request for a snapshot of a volume (like pvc)
volumesnapshotcontents.snapshot.storage.k8s.io						-		think of this as the pv

volumegroupsnapshotclasses.groupsnapshot.storage.k8s.io
volumegroupsnapshots.groupsnapshot.storage.k8s.io
volumegroupsnapshotcontents.groupsnapshot.storage.k8s.io




Components:

1. snapshot controller (aka common snapshot controller)

watches for create/update/delete on volumesnapshot, volumesnapshotcontent, volumegroupsnapshot, volumegroupsnapshotcontent

k8s.gcr.io/sig-storage/snapshot-controller


2. csi snapshotter (aka external-snapshotter)

watches for create/update/delete only for volumesnapshotcontent and volumegroupsnapshotcontent

k8s.gcr.io/sig-storage/csi-snapshotter						

CSI external-snapshotter sidecar talks to CSI over socket (/run/csi/socket)


3. Snapshot conversion web-hook

The snapshot conversion webhook is an HTTP callback which responds to conversion requests,
allowing the API server to convert between the VolumeGroupSnapshotContent v1beta1 API to and from the v1beta2 API.

k8s.gcr.io/sig-storage/snapshot-conversion-webhook


4. CSI Endpoint



Process:

client ---- submits ---> VolumeSnapshot -- to --> API Server
Snapshot Controller --- sees ---> VolumeSnapshot --- and creates ----> VolumeSnapshotContent object (bound to the volumesnapshot)

csi-snapshotter --- watches ---> VolumeSnapshotContent resource ----- triggers CreateSnapshot/DeleteSnapshot ------> CSI Endpoint (CSI driver)

CSI Driver ---- creates the actual snapshot on -----> Storage System

CSI Driver ---- informs ----> csi-snapshotter ------ which informs ---> Snapshot Controller ------ which updates readyToUse on ----> VolumeSnapshot




### Commands

I want to see the available csi drivers
kubectl get csidrivers


What nodes have csi drivers and which csi drivers
k get csinode
k get csinode <name> -o yaml


I want to see the available storage classes
kubectl get storageclass 


The available volumesnapshotclasses
kubectl get volumesnapshotclass




## CLOUD PROVIDER INTERFACE




## CLOUD CONTROLLER MANAGER

This component’s purpose is to run cloud-specific controller loops and to execute cloud-based API calls
Just like the Kube Controller Manager this component manages 'several' controllers, each of which is managing a specific set of resources.

think provisioning an IP address or storage volume on AWS or Azure or Google Cloud or vSphere or OpenStack and so on
the CCM generically utilizes functionality from any vendor implementing the CPI 
When running Kubernetes on a cloud platform, Kubernetes interacts directly with the public or private cloud APIs, and the CCM executes the majority of those API calls. 







# PART II - ADDITIONAL COMPONENTS THAT CAN BE ADDED



## CLUSTER LEVEL LOGGING


## WEB UI


## DNS


## CONTAINER RESOURCE MONITORING


## METRICS SERVER





# PART III - GENERAL KUBERNETES CONCEPTS


## POD

Pod = IPC Linux Namespace { Network Namespace { CGroup { PID of container }, CGroup { PID of container } } }
Pod = PID Namespace + network namespace + IPC namespace + cgroup namespace + mnt namespace + user namespace







## CONTAINERS




## PAUSE CONTAINER

a placeholder container run by the kubelet while the kubelet creates namespaces and cgroups for a container to run inside of



## POD SECURITY STANDARDS


### How it works (I THINK)

A PSS profile on a namespace that enforces what pods can/can't do
securitystandards on pods/containers that show what pods actually need
The PodSecurity Admission controller sees the request for a workload come in, 
looks at the profile on the namespace
compares it with what is on the workload resource
and accepts or denies the request


### Profiles, Controls, Polices, Modes, Versions

The pod security standards are enforced by the PodSecurity Admission Controller.

What are PodSecurity standards?
First think about it this way - is there anything in the PodSpec of a workload that will be allowed or prevented from running.

The standard is made up of profiles. Profiles --have--> controls and controls --have--> policies. 
A policy can be set at Pod level (found directly under .spec.securityContext) OR Container Level (found under .spec.containers[*].securityContext)
A policy just allows a particular .spec to run, e.g. spec.hostNetwork or spec.containers[*].securityContext.capabilities.add

Profiles
	Privileged			-	Unrestricted policy, providing the widest possible level of permissions. This policy allows for known privilege escalations.

	Baseline			-	Minimally restrictive policy which prevents known privilege escalations. Allows the default (minimally specified) Pod configuration.

	Restricted			-	Heavily restricted policy, following current Pod hardening best practices.

Controls
	HostProcess						-	mostly to do with Windows, we ignore for now
	Host Namespaces					-	sharing the host namespace
	Privileged Containers			-	Privileged Pods disable most security mechanisms
	Capabilities					-	Adding additional capabilities 
	HostPath Volumes	
	Host Ports	
	Host Probes/Lifecycle Hooks	
	AppArmor						-	this is a Linux Kernel Security module that allows the ability to restirct programs capabilities with per-program profiles. 
										Capabilities can be network access, raw socket access, permission to rwx files on matching paths, etc.
	SELinux	
	/proc Mount Type	
	Seccomp							-	the ability to allow/restrict system calls to the kernel from a container
	Sysctls

	Volume Types
	Privilege Escalation
	Running As Non Root
	Running AS User


The 3 profiles (privileged, baseline, restricted) can actually be run in one of 3 modes
	enforce			-	reject pods with policy violations
	audit			-	allow pods with policy violoations but includes an audit annotation in the audit log event record
	warn			-	allows pods with policy violations but warns users


The mode can be specified to a version
	<version>		-	a specific kubernetes version, e.g. 1.33.4
	latest			-	the latest version of the profile


Ok, so what is the actual setting for each profile? 
This page has the full list - https://kubernetes.io/docs/concepts/security/pod-security-standards/


### Configure a profile on a namespace:

Add this label to the namespace
pod-security.kubernetes.io/<mode>=<profile>
pod-security.kubernetes.io/<mode>-version=<version>				# version is either a kubernetes version OR latest

For e.g. 	
a setting of ‘kubectl label namespace project1 pod-security.kubernetes.io/enforce=privileged' 			- will allow pods to run with a privileged profile in the project1 namespace
a setting of ‘kubectl label namespace project1 pod-security.kubernetes.io/enforce-version=latest' 		- will enforce the latest version of the policy


### Setting the SecurityContext on a pod or container

The entire reference. Change version as needed
NOTE: https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.32/#securitycontext-v1-core


Key Settings:

pod/SecurityContext
  runAsNonRoot: true or false
  runAsUser: <userID>
  runAsGroup: <groupID>
  fsGroup: <groupID>
  supplementalGroups: []

container/SecurityContext
  runAsNonRoot: true or false
  privileged: true or false											the container gains unrestricted access to the host kernel, bypassing container isolation mechanisims (crgoups, namespaces, etc)
																	retains all linux capabilities (even if explicitly dropped)
																	get direct access to host devices (e.g. /dev, raw sockets), can modify kernel parameters, load kernel modules
  allowPrivilegeEscalation: true or false							a security context setting that controls whether a process can gain more privileges than its parent process
																	becomes true if 'privileged: true' securityContext setting has been set OR CAP_SYS_ADMIN capability is set
  capabilities:
  	drop: ["<list of one or more keywords>" | "ALL"]
  readOnlyRootFilesystem: true or false
  seccompProfile: 
    type: "RuntimeDefault" or "Localhost" or "Unconfined"



## IPAM

Kubernetes IPAM (IP Address Management) refers to the IP address management within a cluster.
Think about how a pod gets an IP address, a subnet mask and default gateway and how it gets DNS.
Same goes for Service.
IPAM can be fulfilled through kube-proxy OR through 3rd party providers/plugins.





## ASSIGNING PODS TO NODES (TAINTS, NODENAME, NODESELECTOR, AFFINITY, TOLERATIONS)


### Settings on a node

taint							-		this is a node repelling a set of pods
	NoSchedule					-		pod cannot be scheduled to node unless it has a matching toleration, ```kubectl taint nodes node1 key1=value1:NoSchedule[-]```
	NoExecute
	PreferNoSchedule



### Settings on a workload

nodeName						-		rigid, select a node based on name


nodeSelector					-		flexible, select a node based on label


affinity
    nodeAffinity
	podAffinity					-		these rules attract pods to a node that matches the rules
	podAntiAffinity				-		repels pods from nodes



tolerations						-		allow the scheduler to schedule pods with matching taints
	e.g. for NoSchedule
	tolerations:
	- key: "key1"
	  operator: "Equal"
	  value: "value"
	  effect: "NoSchedule"
	# OR
	- key: "key1"
	  operator: "Exists"
	  effect: "NoSchedule"



### Commands

I want to see the taints on a node
k describe node <node>





## PKI (CERTIFICATES)

Think about 3 levels
- Cluster Root CA
- Signers (CA's that issue certificates)
- Client Certs and Server Certs for the various components of the cluster (e.g. api server, scheduler, etc.)

NOTE: If kubeadm was not used to deploy the cluster then the CA's and Signers could be different from those noted below


LINK: https://kubernetes.io/docs/setup/best-practices/certificates/


### Cluster Root CA

I want to see the cluster root ca certificate:
1. ls /etc/kubernetes/pki/ca.crt				- do this from a control plane node (this is kubeadm installation specific)
2. k exec -it <pod> -- cat /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
3. kubectl config view --raw -o go-template='{{index ((index (index .clusters 0) "cluster")) "certificate-authority-data"|base64decode}}'



### Signers (CA's)

I want to see all the Certificate Authorities (signers) on the cluster (we can submit a csr to a signer):
From: https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/
kubernetes.io/kube-apiserver-client				-	signs certs that will be honored as client certs by the API Server, never auto approved by the kube-controller-manager
kubernetes.io/kube-apiserver-client-kubelet		-	signs certs that will be honored as client certs by the API Server, maybe auto approved by the kube-controller-manager
kubernetes.io/kubelet-serving					-	signs serving certificates that are honored as a valid kubelet serving certificate by the API server, never auto-approved by kube-controller-manager.
kubernetes.io/legacy-unknown

NOTE: The kube-controller-manager implements control plane signing for each of the built in signers



### Client Certs and Server Certs

API Server
	Server Cert		-	the server cert (think DNS names and IP addresses the cert is valid for)
	Client Cert		-	auth to ETCD

Controller Manager
	Client Cert		-	auth to API Server

Scheduler
	Client Cert		-	auth to API Server

Kube-Proxy/Node
	Client Cert		-	auth to API Server

ETCD 
	Server Cert
	Peer Cert

Kubelet
	Server Cert
	Client Cert		- 	auth to API Server

User
	Client Cert		- 	auth to API Server


### Private/Public Key for signing service accounts

Other than the above certs there is a public/private key pair used for signing service accounts


## AUTHENTICATING WITH KUBERNETES

authentication with a cluster can be by:
client-certificate + client key
username + password
bearer token
external provider (aka proxy)





## SYSTEM NAMESPACES

default
kube-system				-		contains kubeadm-config
kube-public
kube-node-lease			-		should show the heartbeats of a nodes kubelet to the api server






## FEATUREGATES








## USERS AND GROUPS

Users:
```shell
k get clusterrolebindings.rbac.authorization.k8s.io -o yaml | grep -A1 'kind: User'
```

system:kube-controller-manager
system:kube-scheduler
system:kube-proxy
system:kube-scheduler

and then ....
system:node:<nodename>



Groups:
```shell
k get clusterrolebindings.rbac.authorization.k8s.io -o yaml | grep -A1 'kind: Group'
```

system:masters
system:bootstrappers:nodes
system:nodes
system:authenticated
system:unauthenticated
system:monitoring
system:serviceaccounts

Take note of this and delete it later
kind: Group
name: oidc:lxadmsu


ServiceAccounts:


### Commands

```shell
k auth whoami
```



## POD DISRUPTION BUDGET




## RBAC

### Full list of verbs

VERB					HTTP			MEANING
get									-	allow the user to retrieve the state of kubernetes resources
list								-	allow the user to retrieve a list of kubernetes resources
watch								-	allow the user to receive notifications when the state of resource changes
create								-	allows the user to create a new Kubernetes resource
update					PUT			-	allows the user to update an existing Kubernetes resource
patch					PATCH		-	allows the user to make partial updates to an existing Kubernetes resource
delete								-	allows the user to delete an existing Kubernetes resource
deletecollection					-	allows the user to delete a collection of Kubernetes resources
proxy								-	allows the user to access the kube-api-server through a proxy
connect								-	allow the user to connect to the console of a container
redirect							-	allow the user to redirect traffic to a kubernetes service
portforward							-	allow the user to forward network traffic to a kubernetes pod



### Commands

```shell
# Check if a user in a group has access
k auth can-i <verb> <resource> --as=<user> --as-group=<group> -n <namespace>
k auth can-i create backups.velero.io --as=oidc:gazb --as-group=oidc:opsk8s-admins -n infra-backups

```


## NETWORKING

4 levels of network

The network for the pods

The network for the services

The IP addresses of the nodes (the external network)

Other external networks


## KUBERNETES PROVISIONING FOR A POD

### Environment variables


### Files
/var/run/secrets/kubernetes.io/serviceaccount/token			-		a token for the serviceaccount of the pod
/var/run/secrets/kubernetes.io/serviceaccount/ca.crt		-		the Cluster Root CA cert for the kubernetes cluster


# PART IV - CLUSTER CONFIGURATION 

cluster-cidr					-	The IP address range for the pods in the cluster
podCIDR 						-	The CIDR range for the pods on a per node basis
service-cluster-ip-range		-	The IP address range for services in the cluster


### Commands

```shell
# What is the UUID of the cluster
kubectl get ns kube-system -o=jsonpath='{.metadata.uid}'


# I want the entire cluster configuration
kubectl cluster-info dump
```



# PART V - BINARIES AND LOCATIONS



## BINARIES

kubeadm					-		
kubectl


## IMPORTANT FILES AND DIRS ON A NODE

/etc/kubernetes/admin.conf
/etc/kubernetes/kubelet.conf
/etc/kubernetes/bootstrap-kubeconfig
/etc/kubernetes/kubeconfig-kubelet
/etc/kubernetes/kubelet.yaml
/etc/kubernetes/manifests
/etc/kubernetes/pki


/var/log/containers/*.log		-		logs of containers
/var/log/audit/kube/*.log		-		records of all API requests made to the Kubernetes API server


/var/lib/cni
/var/lib/containerd
/var/lib/etcd
/var/lib/kubelet


Manifests:
kubeadm-config			-		contains ClusterConfiguration object, it is read during ‘kubeadm join’, ‘kubeadm reset’ and ‘kubeadm upgrade’



# PART VI - DEBUGGING


## Pod .status

kubectl get -n <namespace> pod/<pod_name>  -o yaml | yq .status

.status.phase		
where is the pod in its lifecycle
  Pending
  Running
  Succeeded
  Failed								
	
.status.conditions[]		
each entry could have one of these condition types (.status.conditions[].type)
  PodScheduled
  Initialized
  PodReadyToStartContainers
  ContainersReady
  Ready
  Unschedulable
AND
each entry could have one of these statuses (.status.conditions[].status)
  true
  false

if the .status.conditions[].status 	-	is false then there will be a .status.conditions[].reason explaining why it is false
AND
.status.containerStatuses[] 		- 	in here will be .status.containerStatuses[].state.waiting.reason - this is where CrashLoopBackOff will be seen



## Commands

```shell
# I want to run a debug pod on a node
kubectl debug -n kube-system -it --image alpine node/<node>


# I want to debug a container in a pod (using superdebug)
curl -o ~/.local/bin/kubectl-superdebug https://raw.githubusercontent.com/JonMerlevede/kubectl-superdebug/refs/heads/main/kubectl-superdebug
chmod +x ~/.local/bin/kubectl-superdebug
 
kubectl-superdebug -p <pod_to_debug> -t <container_in_pod_to_debug> -n <namespace> --context <context_from_kube_config> --image alpine -C /bin/bash
 
Retrieving pod spec
Checking for existing ephemeral containers
Starting kube proxy
Waiting for proxy to come up...
Starting to serve on 127.0.0.1:8001
Patching pod fluent-bit-64gc2, creating ephemeral container debug-nbldbq
Created ephemeral debug container. Give it some time to start, then connect to it using the following command:
 
kubectl -n kube-system attach fluent-bit-64gc2 -i -t -c "debug-nbldbq"
 
You may disconnect from the debug container by pressing Ctrl-P followed by Ctrl-D.
If you exit the shell, the debug container will be terminated.
Stopping kube proxy


# A way of running an aribitary container attached to node namespace and connecting as root
## Create debug profile
cat <<___EOT >debug-profile.yaml
securityContext:
  runAsUser: 0
___EOT
 
## Create debug pod
kubectl debug -n kube-system -it --profile sysadmin --image aplregistry.aarnet.edu.au/hub.docker.com/rook/ceph:v1.16.3 node/nsw-artm-opsk8stest-ceph1 --custom debug-profile.yaml -- bash


# Namespace stuck in terminating
## try it this way
kubectl api-resources --verbs=list --namespaced -o name | xargs -n 1 kubectl get --show-kind --ignore-not-found -n <namespace>
# then remove the finalizers on the pending resources

# OR

## try it this way
for ns in $(kubectl get ns --field-selector status.phase=Terminating -o jsonpath='{.items[*].metadata.name}')
do
  kubectl get ns $ns -o json | jq '.spec.finalizers = []' | kubectl replace --raw "/api/v1/namespaces/$ns/finalize" -f -
done


# Get all pods on a specific node
kubectl get pods -A -o wide --field-selector spec.nodeName=sa-prka-wrke2dev1


# Get all pods not in state running
kubectl get pods -n cilium-cni --field-selector "status.phase!=Running"


# Check container images used in a namespace
kubectl -n <namespace> get pods -o jsonpath="{.items[*].spec['initContainers', 'containers'][*].image}" | tr -s '[[:space:]]' '\n' | sort | uniq -c


# Drain a node
kubectl drain --ignore-daemonsets --delete-emptydir-data --force --disable-eviction=true --grace-period=5 --skip-wait-for-delete-timeout=5 nsw-etca-opsk8stest-iwrk1


# Check resource usage on a node
for node in $(kubectl get nodes -o name); do echo $node; kubectl describe ${node} | grep Allocated -A5; echo; done   
```

# PART VII - KUBECONFIG


## Kubeconfigs in the cluster

the following components need to communicate with the api-server and will therefore also need
some kind of kubeconfig
kubelet
controller-manager
scheduler


## User Kubeconfig (generally referred to as admin.conf)

kubectl will find kubeconfig file by:
$KUBECONFIG						-		colon delimited list of kubeconfig files, kubectl merges the files into an effective configuration (the list is semicolon delimted on windows). earlier file settings win over subsequent file settings
$HOME/.kube/config
--kubeconfig


kubectl will determine context to use by:
--context						-		namespace/cluster-server-url-with-dashes/username, e.g. default/api-crc-testing:6443/kubeadmin
use the current-context value from config






### Commands

I want to see the entire config
kubectl config view




# PART VIII - KUBECTL

I want to see the options/flags available to the kubectl command
```shell
kubectl options
```

I want to set the context while using kubectl command
```shell
kubectl --context <context> .....
```

Log level verbosity
```shell
kubectl -v=n
```

I want to see the definition for a resource without going to the website for the resource reference
```shell
kubectl explain <resource> |--recursive  less
```


kubectl get pods --all-namespaces \
-o jsonpath='{range .items[*]}{"\n"}{.metadata.name}{":\t"}{.metadata.namespace}{":\t"}{range .spec.containers[*]}{.image}{", "}{end}{end}' | grep <your-image-name>
