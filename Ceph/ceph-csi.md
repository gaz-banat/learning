
# CEPHFS

## csidriver

cm/ceph-config                                          # think about this as how the ceph cluster is configured
cm/ceph-csi-config                                      # think about this as how ceph-csi should be configured to talk to the ceph cluster, each cluster is an array item
                                                        # the clusterID + monitors, the rbd, cephfs, nfs settings, the readAffinity setting 
cm/ceph-csi-encryption-kms-config

### csi-cephfs-plugin-provisioner
sa, cr, crb, role, rb for provisioner                   # rbac for provisioner
deployment/csi-cephfs-plugin-provisioner                # i believe this provisioner is also a 'controller' and is watching for pvc requests to the api

  containers
        csi-cephfsplugin 
        csi-provisioner                                 # listening for events that create pvc
        csi-resizer
        csi-snapshotter
        liveness-prometheus


### csi-cephfs-plugin-provisioner
sa, cr, crb, role, rb for node plugin                   # rbac for node plugin
ds/csi-cephfsplugin                                     # one for each node, kubelet calls this to mount the subvolume into the app

   containers
        csi-cephfsplugin
            Starting driver type: cephfs with name: cephfs.csi.ceph.com
        driver-registrar
        liveness-prometheus 


secret/<provisioner-secret>
secret/<node-stage-secret>
secret/<controller-expand-secret>

NOTE: the above 3 could be in one secret



## sc/cephfs.csi.ceph.com

The storage class will
 
    1. have a reclaim policy, allowvolumeexpansion and have a volumebinding mode

    2. know the filesystem and pool to use
    
    3. know the provisioner to use                  -   how is the provsioner determined? does the namespace have to be used in calling the provisioner

    4. know the cluster id

    5. provisioner-secret                           -   adminID and adminkey  (it invokes teh provisioner with this secret to talk to the storage cluster)

    6. node-stage-secret                            -   i think this is to mount the volume into the node and make available to the app

    7. controller-expand-secret                     -   what is this for?

    8. fstype


## Mounting 

Mounting of a cephfs volume to a node can happen via 

kernel 

fuse 
a way for non-priv users to create a filesystem without editing kernel code 





# CEPH-RBD




----------

# COMMANDS TO PUT IN THE SECRETS

## DEV
vault kv put secret/k8s/dev/ceph/csi-ceph-rbd-provisioner userID=csi-rbd-provisioner userKey=AQCySYhnhF2jJRAA0r6JL8akAUPXwhzmASMRPg==

vault kv put secret/k8s/dev/ceph/csi-ceph-rbd-node userID=csi-rbd-node userKey=AQCySYhnu4rXMxAAJQgn2WexasmLzKawqVVS+Q==

  vault kv put secret/k8s/dev/ceph/csi-rbd-opsk8sdev userID=csi-rbd-opsk8sdev userKey=AQAaT4hnTUm4KRAAuE5Lyn63v/sBujOQW+od0g==



vault kv put secret/k8s/dev/ceph/csi-cephfs-node adminID=csi-cephfs-node adminKey=AQCzSYhnF2fSFBAA5uc/pQZxwGw77gmUNHj27Q==

vault kv put secret/k8s/dev/ceph/csi-cephfs-provisioner adminID=csi-cephfs-provisioner adminKey=AQCzSYhnce6VBhAAhMIjGdzthrCSIoia30o6EA==

  vault kv put secret/k8s/dev/ceph/csi-cephfs-opsk8sdev adminID=csi-cephfs-opsk8sdev adminKey=AQAEfI1nnuKMNhAAZ5hYUPB+JAmy73YEGn20Eg==



## LAB

vault kv put secret/k8s/lab/ceph/csi-ceph-rbd-provisioner userID=csi-rbd-provisioner userKey=AQCySYhnhF2jJRAA0r6JL8akAUPXwhzmASMRPg==

vault kv put secret/k8s/lab/ceph/csi-ceph-rbd-node userID=csi-rbd-node userKey=AQCySYhnu4rXMxAAJQgn2WexasmLzKawqVVS+Q==

  vault kv put secret/k8s/lab/ceph/csi-rbd-opsk8slab userID=csi-rbd-opsk8slab userKey=AQC9fY1n/krJCBAAX205/F+ZgFH0lY+ZOW57dw==





vault kv put secret/k8s/lab/ceph/csi-cephfs-node adminID=csi-cephfs-node adminKey=AQCzSYhnF2fSFBAA5uc/pQZxwGw77gmUNHj27Q==

vault kv put secret/k8s/lab/ceph/csi-cephfs-provisioner adminID=csi-cephfs-provisioner adminKey=AQCzSYhnce6VBhAAhMIjGdzthrCSIoia30o6EA==

  vault kv put secret/k8s/lab/ceph/csi-cephfs-opsk8slab adminID=csi-cephfs-opsk8slab adminKey=AQAqfo1nI6rBEhAATN3VdbCNBHr8/R8j5ur6Ww==