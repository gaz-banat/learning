
# OVERVIEW

Think of Rook as composed of 3 parts
1. The CRD's
2. The rook-ceph operator                   -   operator pod, csi pods, discover pods, 
3. The rook-ceph cluster                    -   toolbox pod,  mon pods, mgr pods, rgw pods, prep osd pods, osd pods
                                                resources for monitoring, dashboard, ingress
                                                cephblockpools + storage classes for the cephblockpools
                                                cephfilesystems + storage classes for the cephfilesystems
                                                cephobjectstores + storage classes for the cephobjectstores + ingresses for cephobjectstores



# CRD's

Documentation for CRD's              -       https://rook.io/docs/rook/latest-release/CRDs/Cluster/ceph-cluster-crd/ (and then find your way accordingly)


The following CRDs come from the crds bundle in deploy/examples/crds.yaml:

cephclusters.ceph.rook.io

cephblockpools.ceph.rook.io                             -    a pool to be used for rados block devices
cephblockpoolradosnamespaces.ceph.rook.io               -    think of it as a segregation in a cephblockpool

cephrbdmirrors.ceph.rook.io


cephfilesystems.ceph.rook.io                            -    a cr for creating a ceph filesystem (remember that a ceph filesystem is represented as a volume - `k rook-ceph ceph fs volums ls`)
                                                             (underlying data and metadata pools are created automatically)
cephfilesystemsubvolumegroups.ceph.rook.io              -    think of this as a group, into which cephfs subvolumes will be put, the group resource will be used for policies
cephfilesystemmirrors.ceph.rook.io


cephobjectstores.ceph.rook.io                           -   Object storage exposes an S3 API and or a Swift API to the storage cluster for applications to put and get data
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



The following CRDs come from the installation of the operator:

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


Deployment/ceph-csi-controller-manager  -       quay.io/cephcsi/ceph-csi-operator, CSI Driver


Command - ```helm template rooky rook-release/rook-ceph --dry-run | grep ^kind```

The rook-ceph-operator then creates the csidriver workloads:
- deployment/csi-cephfsplugin-provisioner and ds/csi-cephfsplugin
- deployment/csi-rbdplugin-provisioner and ds/csi-rbdplugin



## Components in the rook-ceph operator (not containers, think software components)

controller
ceph-operator-config-controller
ceph-object-store-user-controller
clusterdisruption-controller


## Configuration for the operator

```shell
k get cm rook-ceph-operator-config -o yaml -n rook-ceph
```


## How are the csidrivers structured

`the main driver is this (providing controller service and identity service) - quay.io/cephcsi/cephcsi`

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
StorageClasses
CephCluster
CephBlockPool
CephFilesystem
CephFilesystemSubVolumeGroup
CephObjectStore

Command - ```helm template rooky rook-release/rook-ceph-cluster --dry-run | yq '. | (.kind, .metadata.name)'```



## This is where rook meets ceph

### cephfs
When you create a cephfilesystem resource:
 - a cephfs volume is created (k rook-ceph ceph fs volume ls)at least 
 - 2 pools are created in the ceph cluster - the metadata pool, the data pool (k rook-ceph ceph osd lspools)

then a pvc in a namespace referring the ceph-filesystem storage class
gives birth to a pv in the kubernetes system
which creates a subvolumegroup in the cephfs volume created above
and then creates a subvolume in the subvolume group

so - the pv is a subvolume


### rbd
When you create a cephblockpool resource:
 - an rbd pool is created (k rook-ceph ceph osd lspools)

then a pvc in a namespace referring the ceph-block storage class
gives birth to a pv in the kubernetes system
which creates an "image" in the rbd pool

so - the pv is an image


### objectstore
When you create a cephobjectstore resource: 
- at least 7 pools are created in the ceph cluster - control, meta, log, buckets.index, buckets.non-ec, otp, data (k rook-ceph ceph osd lspools)


CRUSH rules come into play by marking nodes with labels - topology.kubernetes.io/region, toplogy.kubernetes.io/zone




# WORKINGS


## The Operator reads the CephCluster resource and provides:

confgimap/rook-ceph-csi-config        -       configuration for the CSI driver, a list of ceph clusters and their connection details
                                              so when a storageclass calls the csi driver with a particular cluster, the csi driver knows how to get to that cluster




## CEPHFS

### Understanding CephFS Subvolumegroups

The SC provides some key pieces of information for the kubelet when it is invoking the CSI Driver
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

## REMOVE A CEPH CLUSTER

https://rook.io/docs/rook/v1.10/Getting-Started/ceph-teardown/ AND https://www.talos.dev/v1.9/kubernetes-guides/configuration/ceph-with-rook/

```shell
kubectl -n rook-ceph patch cephcluster opsk8stest --type merge -p '{"spec":{"cleanupPolicy":{"confirmation":"yes-really-destroy-data"}}}'

kbeh overlays/opsk8stest | k delete -f -

might have to patch the finalizer

kubectl -n rook-ceph get cephcluster


for node in `kubectl get nodes --no-headers | awk '{print $1}'`; do echo $node; kubectl debug node/${node} --image=alpine -it -- rm -rf /host/var/lib/rook; echo; echo; done
for node in `kubectl get nodes --no-headers | awk '{print $1}'`; do echo $node; kubectl debug node/${node} --image=alpine -it -- ls -l /host/var/lib/rook; echo; echo; done
k get pods -o name | grep node-debugger | sed -e 's|^pod/||' | xargs -n 1 kubectl delete pod 


export node=<node>  # for the OSD nodes

talosctl -n ${node} get volumestatus # note the disk on which the volumes state,meta,ephemeral are located. This disk should be absent in the --user-disks-to-wipe

k drain $node --ignore-daemonsets

talosctl -n ${node}$ reset \
  --user-disks-to-wipe "/dev/nvme0n1,/dev/nvme1n1,/dev/nvme2n1,/dev/nvme3n1,/dev/nvme4n1,/dev/nvme5n1,/dev/nvme7n1,/dev/nvme8n1,/dev/nvme9n1" \
  --wipe-mode user-disks \
  --reboot

# talosctl reboot -n $node

k uncordon $node
```



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