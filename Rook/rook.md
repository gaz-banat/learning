
# OVERVIEW

Think of Rook as composed of 3 parts
1. The CRD's

2. The rook-ceph operator                   
- operator pod (and its associated config via cm/rook-ceph-operator-config)
- csi-cephfsplugin-provsioner pods, csi-cephfsplugin pods, csi-rbdplugin-provisioner pods, csi-rbdplugin pods
- rook-discover pods

3. The rook-ceph cluster                    
- toolbox pod
- mon pods, mgr pods, rgw pods, 
- prep osd pods, osd pods
- resources for monitoring, dashboard, ingress
- cephblockpools + storage classes for the cephblockpools
- cephfilesystems + storage classes for the cephfilesystems
- cephobjectstores + storage classes for the cephobjectstores + ingresses for cephobjectstores
- crash-collector
- ceph-exporter

Where do the following components fit?
job/rook-ceph-detect-version  
job/ceph-file-controller-detect-version 
job/ceph-object-controller-detect-version 
rbdMirrorPeer


# CRD's

Documentation for CRD's                           -       https://rook.io/docs/rook/latest-release/CRDs/Cluster/ceph-cluster-crd/ (and then find your way accordingly)


## The following CRDs come from the crds bundle in deploy/examples/crds.yaml:

cephclusters.ceph.rook.io

cephblockpools.ceph.rook.io                       -    a pool to be used for rados block devices
cephblockpoolradosnamespaces.ceph.rook.io         -    think of it as a segregation in a cephblockpool

cephrbdmirrors.ceph.rook.io

cephfilesystems.ceph.rook.io                      -   a cr for creating a ceph filesystem 
                                                      (remember that a ceph filesystem is represented as a volume - `k rook-ceph ceph fs volums ls`)
                                                      (underlying data and metadata pools are created automatically)
cephfilesystemsubvolumegroups.ceph.rook.io        -    think of this as a group, into which cephfs subvolumes will be put, the group resource will be used for policies
cephfilesystemmirrors.ceph.rook.io


cephobjectstores.ceph.rook.io                     -   Object storage exposes an S3 API and or a Swift API to the storage cluster for applications to put and get data
cephobjectrealms.ceph.rook.io
cephobjectstoreusers.ceph.rook.io
cephobjectzonegroups.ceph.rook.io
cephobjectzones.ceph.rook.io
cephbucketnotifications.ceph.rook.io
cephbuckettopics.ceph.rook.io
objectbuckets.objectbucket.io
objectbucketclaims.objectbucket.io

cephnfses.ceph.rook.io

cephclients.ceph.rook.io

cephcosidrivers.ceph.rook.io



## The following CRDs come from the installation of the operator:

cephconnections.csi.ceph.io

clientprofilemappings.csi.ceph.io
clientprofiles.csi.ceph.io
drivers.csi.ceph.io
operatorconfigs.csi.ceph.io




# ROOK-CEPH (the operator)

## Helm chart for rook operator

https://charts.rook.io/release repo and rook-ceph chart (source code is in rook repo at deploy/charts/rook-ceph/)


The rook-ceph (operator) helm chart provides (amongst many other things like ConfigMap, ServiceAccount, ClusterRole, ClusterRoleBinding, RoleBinding, etc.):

Deployment/rook-ceph-operator           -       docker.io/rook/ceph image, the operator pod  
Deployment/ceph-csi-controller-manager  -       quay.io/cephcsi/ceph-csi-operator, CSI Driver  (WE DONT SEEM TO USE THIS IN AARNET)


Command - ```helm template rooky rook-release/rook-ceph --dry-run | yq (.kind)```

The rook-ceph-operator then creates the csidriver and rook-discover workloads:
- deployment/csi-cephfsplugin-provisioner
- ds/csi-cephfsplugin
- deployment/csi-rbdplugin-provisioner
- ds/csi-rbdplugin
- ds/rook-discover


## Components in the rook-ceph operator (think software components not containers)

cluster-controller
ceph-operator-config-controller
ceph-object-store-user-controller
clusterdisruption-controller
op-mon
op-mgr
op-mds
op-osd
object-controller
file-controller
ceph-csi


## Configuration for the operator

```shell
k get cm rook-ceph-operator-config -o yaml -n rook-ceph
```


## How are the csidriver deployments and pods structured

NOTE: the main driver (providing controller service and identity service) is this image - quay.io/cephcsi/cephcsi

deployment/csi-cephfsplugin-provisioner                       -   the controller component
    pod/xxx-yyy-zzz
      container/csi-cephfsplugin                              -   quay.io/cephcsi/cephcsi
      container/liveness-prometheus                           -   quay.io/cephcsi/cephcsi
      container/log-collector                                 -   quay.io/cephcsi/cephcsi
      container/csi-provisioner                               -   registry.k8s.io/sig-storage/csi-provisioner
      container/csi-attacher                                  -   registry.k8s.io/sig-storage/csi-attacher
      container/csi-snapshotter                               -   registry.k8s.io/sig-storage/csi-snapshotter
      container/csi-resizer                                   -   registry.k8s.io/sig-storage/csi-resizer


ds/csi-cephfsplugin                                           -   the node component
    pod/xxx-yyy-zzz
        container/csi-cephfsplugin                            -   quay.io/cephcsi/cephcsi
        container/liveness-prometheus                         -   quay.io/cephcsi/cephcsi
        container/log-collector                               -   quay.io/cephcsi/cephcsi
        container/driver-registrar                            -   registry.k8s.io/sig-storage/csi-node-driver-registrar


deployment/csi-rbdplugin-provisioner
    pod/xxx-yyy-zzz
      container/csi-rbdplugin    
      container/csi-provisioner
      container/csi-attacher
      container/csi-snapshotter
      container/csi-resizer
      container/liveness-prometheus
      container/log-collector

ds/csi-rbdplugin
    pod/xxx-yyy-zzz
        container/csi-rbdplugin
        container/driver-registrar
        container/liveness-prometheus
        container/log-collector



# ROOK-CEPH-CLUSTER (the cluster) 

## Helm chart for rook cluster
https://charts.rook.io/release repo and rook-ceph-cluster chart (source code is in rook repo at deploy/charts/rook-ceph-cluster/)


The rook-ceph-cluster helm chart provides (amongst other things): 
CephCluster
StorageClasses
CephBlockPool
CephFilesystem
CephFilesystemSubVolumeGroup
CephObjectStore

Command - ```helm template rooky rook-release/rook-ceph-cluster --dry-run | yq '. | (.kind, .metadata.name)'```

## The operator reads the above resources and provisions these workloads and configmap:
rook-ceph-mon
rook-ceph-mgr
rook-ceph-mds

rook-ceph-osdprepare
ceph-file-controller
ceph-object-controller-detect-version

rook-ceph-osd (x N)
rook-ceph-crashcollector (x N)
rook-ceph-exporter (x N)

rook-ceph-tools

rook-ceph-rgw-ceph-objectstore-a (x N)


configmap/rook-ceph-csi-config        
- configuration for the CSI driver
- a list of ceph clusters and their connection details
  so when kubelet calls the csi driver using a storageclass, the csi driver knows how to get to that cluster

  

# WORKINGS

## Daemons

Traditional services of Ceph (aka daemons) continue to be daemons in rook
Some others are added
the total list is:
Mgr
Mon
Osd
rbdMirrorPeer
crashcollector
cephexporter


## Authentication (cephx)

### Entities

Client      -   client entities (those which start with client.)
  admin
  ceph-exporter
  crash collector
  any csi
  any osd-lockbox
  rbd-mirror-peer
  rgw
Service     -   service entities (those that dont start with client.)
  mds
  osd
  mgr


### Rook for configuration purposes divides keys into 2 types

Daemons                   
  Core Daemons (Services) -   mon, mgr, osd, mds
  Child resource daemons  -   client.admin, client.ceph-exporter, client.crash (crash collector)
  gateways                -   client.rgw.* 

Non Daemons               -   csi, cephclient, rbd mirror peer



### Key Locations

within ceph (think server side):

There are 2 places where keys are located
1. The mon auth dbx (ceph auth ls) - all keys except the mon key
2. A filesystem path on each mon node for mon key - 
e.g. ```talosctl read /var/lib/rook/mon-b/data/keyring -n nsw-mcqm-opsk8sdev-mon1```
  
within namespace (think client side):

In the namespace there will be keys
1. either to be accessed by pods that run as daemons (usually with 'keyring' in the name)
2. or for the storage class parameters



### For client.admin

Rook operator sets up a client for admin in the cluster
```shell
k rook-ceph ceph auth get client.admin
```

Then the operator sets up a secret in rook-ceph namespace
```shell
k get secret rook-ceph-admin-keyring -n rook-ceph -o yaml
```
The operator uses this client identity to talk to the ceph cluster


### For CSI pods

Rook sets up a client for the CSI components
```shell
$ k rook-ceph ceph auth ls | grep csi
client.csi-cephfs-node.1
client.csi-cephfs-provisioner.1
client.csi-rbd-node.1
client.csi-rbd-provisioner.1
```

Rook sets up secrets in rook-ceph namespace
```shell
$ k get secret -n rook-ceph | grep csi
rook-csi-cephfs-node                             kubernetes.io/rook   2      3d21h
rook-csi-cephfs-provisioner                      kubernetes.io/rook   2      3d21h
rook-csi-rbd-node                                kubernetes.io/rook   2      3d21h
rook-csi-rbd-provisioner                         kubernetes.io/rook   2      3d21h
```

Rook populates the secretnames into the storage class that the csi provisioner and nodes will use
```shell
$ k get sc ceph-filesystem -o yaml | yq .parameters | grep secret
csi.storage.k8s.io/controller-expand-secret-name: rook-csi-cephfs-provisioner
csi.storage.k8s.io/controller-expand-secret-namespace: rook-ceph
csi.storage.k8s.io/node-stage-secret-name: rook-csi-cephfs-node
csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph
csi.storage.k8s.io/provisioner-secret-name: rook-csi-cephfs-provisioner
csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph
```

The CSI plugin pods (provisioner and node) will read the storage class and 
use the clients name/key in the secret to authenticate with the ceph cluster
and then take pvc/pv related actions



## This is where rook meets ceph

### cephfs
When you create a CephFilesystem resource:
 - a cephfs volume is created (```k rook-ceph ceph fs volume ls```) at least
 - 2 pools are created in the ceph cluster - the metadata pool, the data pool (```k rook-ceph ceph osd lspools```)

When you create a CephFilesystemSubVolumeGroup resource
- a subvolume group is created in the referenced CephFilesystem (```k rook-ceph ceph fs subvolumegroup ls <volume>```)

Then at some stage you create a PVC
- the pvc in a namespace refers to let's say the 'ceph-filesystem' storage class,
- kubernetes gives birth to a pv,
- the pv is a subvolume in the subvolumegroup 
(```k rook-ceph ceph fs subvolume ls <volume> <subvolumegroup>```)
(```k rook-ceph ceph fs subvolume info <volume>  <subvolume> <subvolumegroup>```)

SO - THE PV IS REALLY A SUBVOLUME IN A SUBVOLUMEGROUP OF A CEPH FS VOLUME


### rbd
When you create a cephblockpool resource:
 - an rbd pool is created (```k rook-ceph ceph osd lspools```)

then a pvc in a namespace referring the ceph-block storage class,
gives birth to a pv in the kubernetes system,
which creates an "image" in the rbd pool

SO - THE PV IS AN IMAGE


### objectstore
When you create a cephobjectstore resource: 
- at least 7 pools are created in the ceph cluster - control, meta, log, buckets.index, buckets.non-ec, otp, data (k rook-ceph ceph osd lspools)


CRUSH rules come into play by marking nodes with labels - topology.kubernetes.io/region, toplogy.kubernetes.io/zone












## CEPHFS

### Understanding CephFS Subvolumegroups

The SC provides some key pieces of information for the kubelet when the kubelet is invoking the CSI Driver
- which ceph cluster to talk to
- what are the parameters to use when talking to the ceph cluster
- which ceph filesystem to use when talking to the cluster

Subvolumegroups are funny in the sense that you need a new StorageClass per subvolumegroup. The storageclass specifies
- a clusterID different to the main ceph clusterID. This different clusterID will help to reach the subvolume group
- the ceph filesystem in which the subvolumegroup resides



### What does a cephfs mount look like on a node

```shell

# delimited by spaces
csi-cephfs-node@e58e6906-e0d4-4362-9b63-e96abf47ef47.ceph-filesystem=/volumes/csi/csi-vol-49cd79a0-9f61-41bf-8afe-4dcfad8b851c/f3a876f5-8eef-4318-b804-6172996f8e59           # Device OR Source
/host/var/lib/kubelet/plugins/kubernetes.io/csi/rook-ceph.cephfs.csi.ceph.com/ab05a29b150022c116e757c71babbe5947ce68e3f62630b6f982319446767e5e/globalmount                    # mount point
ceph                                                                                                                                                                          # fs type
rw,relatime,name=csi-cephfs-node,secret=<hidden>,ms_mode=prefer-crc,acl,mon_addr=10.98.199.99:3300/10.109.233.34:3300/10.109.58.72:3300                                       # mount options
0                                                                                                                                                                             # dump frequency, 0 means dump command will not dump this filesystem
0                                                                                                                                                                             # fsck order value, 0 means no check, root filesystems are typically 1 followed by other file systems
```


## OBJECT STORAGE 

### ObjectBucketClaim

When you provision an S3 bucket into rook-ceph using an objectbucketclaim then 
1. The claim will cause a bucket to be created
2. A secret will be created in the namespace of the objectbucketclaim - the secret will have the AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY 
3. A cm will be created in the namespace of the objectbucketclaim - the cm will have the url, bucket name and port for the connection




# HOW TO


## CREATE A BUCKET WITH A CUSTOM USER, ACCESS KEY AND SECRET ACCESS KEY

```shell
## NOTE: THERE IS NOW A SCRIPT IN ADMINSTRATION REPO - tools/create-rgw-user_and_bucket.sh TO ACCOMPLISH THIS

# Declare vars
cluster=opsk8s
user=gazb

## Generate accessKey and secretKey
accessKey="ZBO7NP2AVZJY4G03GBCK"
secretKey="2D07pMWJTcMl8Fk3Wk6Py1SWBYTrtjsX2HVWejSz"


## Create a secret in vault with accesskey and secretKey
vault kv put secret/k8s/$cluster/rook-ceph/users/$user accessKey=$accessKey secretKey=$secretKey


## Create the external secret which will pull the above vault secret
---
apiVersion: external-secrets.io/v1
kind: ExternalSecret
metadata:
  name: ${user}-s3-keys
  namespace: rook-ceph
spec:
  refreshInterval: 15m
  secretStoreRef:
    kind: ClusterSecretStore
    name: vault-backend-global
  dataFrom:
    - extract:
        key: k8s/$cluster/rook-ceph/users/$user


## create the cephobjectstoreuser
---
apiVersion: ceph.rook.io/v1
kind: CephObjectStoreUser
metadata:
  name: $user
  namespace: rook-ceph
spec:
  clusterNamespace: rook-ceph
  store: ceph-objectstore
  displayName: "$user"
  keys:
    - accessKeyRef:
        name: ${user}-s3-keys
        key: accessKey
      secretKeyRef:
        name: ${user}-s3-keys
        key: secretKey
  quotas:
    maxBuckets: 1000
    maxSize: 1Ti 
    maxObjects: -1

## create the objectbucketclaim (this uses the cephobjectbucketuser from above)
---
apiVersion: objectbucket.io/v1alpha1
kind: ObjectBucketClaim
metadata:
  name: $user
  namespace: rook-ceph
spec:
  bucketName: $user
  storageClassName: ceph-bucket
  additionalConfig:
    bucketOwner: $user
    maxSize: "100Gi"
```



## Show CSI images being used in cluster
```shell
kubectl --namespace rook-ceph get pod -o jsonpath='{range .items[*]}{range .spec.containers[*]}{.image}{"\n"}' -l 'app in (csi-rbdplugin,csi-rbdplugin-provisioner,csi-cephfsplugin,csi-cephfsplugin-provisioner)' | sort | uniq
```


## Get the versions of the components in use
```shell
kubectl -n rook-ceph get deployments -l rook_cluster=rook-ceph \
-o jsonpath='{range .items[*]}{.metadata.name}{"  \treq/upd/avl: "}{.spec.replicas}{"/"}{.status.updatedReplicas}{"/"}{.status.readyReplicas}{"  \trook-version="}{.metadata.labels.rook-version}{"\n"}{end}'
```