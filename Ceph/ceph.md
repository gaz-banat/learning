

# CONCEPTS

## RADOS
Reliable Autonomous Distributed Object Store
The foundation of Ceph, managing data as objects distributed across the cluster.


## LIBRADOS
A library allowing apps to directly access RADOS with support for varions languages (C, C++, Java, Python, Ruby and PHP)


## Rados Object 
has an identifier + binary data + metadata (kv pairs)
identifier (id) is unique across the entire cluster 

NOTE: CephFS uses a separate metadata pool to store information about a file like owner, created date, last modified date, etc.


## Storage Interfaces/Types
RADOSGW								-	A REST Gateway to a RADOS Object Store. This is bucket based and compatible with S3 and Swift. Depends on LIBRADOS
RADOS Block Device (RBD)			-	A RADOS Object store presented as a block device to the consumer. Depends on LIBRADOS
CephFS								-	A RADOS Object store presented as a filesystem to the consumer

NOTE: 
rbd is a disk, so good for small random IO workloads
cephfs is a filesystem, so better suited for large sequential reads/writes and multi-user shared file environments

## Daemons
Monitor (mon)						-	Maintain the cluster map and ensure consensus. (v2 is on port 3300, v1 is on port 6789)
										Provides copies of the cluster map to storage cluster clients
Manager (mgr)						-	Track runtime metrics (monitoring) - collect performance data resource utilization, system load, disk/cpu performance
										manage cluster state information
										Hosting Modules	- provide interface to external management systems (Web GUI dashboard, REST API, metric exporter like prometheus) by hosting python modules
										state of the cluster = monitor map + OSD map + PG map + MDS map + CRUSH map
Object Storage Daemon (osd)			-	Store data on physical disks and handle replication, erasure coding, and recovery.
Meta Data Server (mds)				-	CephFS relies on Metadata Servers (MDS) for all metadata operations (file creation, deletion, resolve directory paths, attribute changes, inode information)
Ceph Object Gateway (rgw)			-	provides a RESTful gateway between apps and Ceph Storage Clusters
										built on librados, S3 compatible API available, Swift compatible API available,
										User - uses the S3 interface
										Subuser - uses the Swift interface

(Not sure where i got the below from)										
AlertManager
Ceph-Exporter
CrashCollector
Node-Exporter
Prometheus


NOTE: there is no daemon for RBD!!



## CRUSH (Controlled Replication Under Scalable Hashing)
This is the algorithm that Ceph uses to compute storage locations for objects so it can decide the placement group for a RADOS object
CRUSH determines data placement and retrieval without a central lookup table enabling scalability and dynamic rebalancing.
CRUSH can identify OSDs by storage media type and organize OSDs hierarchically into regions, zones, nodes, racks and rows.


A CRUSH ruleset on the other hand applies to a pool. Administrators set the CRUSH ruleset when creating a pool. 
A CRUSH Ruleset determines 
- failure domains (region, zone, node, rack, row)
- performance domains for a pool 



## Pool 
A logical partition, think of it as being used for separation
Also think of it as a collection of storage volumes
Characteristics of pools
	- Pools are either “replicated” or “erasure coded” for the purpose of data protection.
	- replicated size
	- minimum size
	- autoscale mode
	- crush rule associated with pool
	- flags
	- application associated with the pool

Pools get associated with applications before they can be used


## Device 
A disk on which a PG will be put
a device can have
- an ID 
- a CLASS				-		nvme, hdd, sdd
- a Type				-		root
- a name


## Storage Volume (STILL NOT SURE ABOUT THIS)
The basic unit of storage, e.g. allocated space on a disk, a single tape cartridge



## Placement Group (PG)
Pools manage data by sharding them into placement groups
A PG is a subset of a pool that serves to contain a collection of objects
A placement group can be on one or more OSD's
System administrators set the placement group count when creating or modifying a pool.

NOTES: 300 PG per OSD is default




## How it all hangs together


							Client (the client provides pool name, user and secret key)
							1.|			  2.|		  3.|
							  |				|			|
							  |				|			|
							  |				|		RADOS Object (client reads or writes a rados object)
							  |				|			|
							  |				|			|
	CEPHX (the authentication protocol)		|			|
							  |				|			|
							  |				|			|
MONITORS (daemon)			  |				|			|								MANAGER (daemon)
			Monitor 1		  |				|			|
							  |				|			|
			Monitor 2 --------				|			|
											|			|
			Monitor 3						|			|
							  				|			|
											|			|
MDS (daemon) -------------------------------			|		  
														|
														|
														|
							   -------------------------	
STORAGE INTERFACES			  |													 
							CephFS 						RBD						Object
							  |
							  |
							  |
							  |
							  |
						ceph-filesystem																(the "volume" as provisioned from rook 'ceph fs volume ls' created because of a CephFilesystem resource)
						 	csi																		(the subvolumegroup 'cephf fs subvolumegroup ls')
							  |																		
							  |  
							  |
  	  					 C R U S H	  																(i think CRUSH is here)
						 	  |
							  |
							  |
							  |
OSD (daemon)	OSD		OSD	  OSD	OSD		OSD 
							  |
							  |----------------------------------
							  |									|
							  |									|
  Pool 						  |									|
    ceph-blockpool		ceph-filesystem-data0	ceph-filesystem-metadata 
		image				  |
		image				  |
		image				  |
							  |
							  |
  osd instances				  V							  
	osd.1		osd.2		osd.3		osd.4		osd.5											(this is a case of one osd per device, there could be multiple osd's per device!)
	(device)  	(device)  	(device)  	(device)  	(device)

	PG1			PG1			PG3			PG3			PG4
	PG2			PG2			PG4			PG2			PG5
	PG5			PG5			PG1			PG4 		PG3 
	... 		...			...			...			...


In the above 
- we are showing the use of the CephFS storage interface
- the data pool is called ceph-filesystem-data0 and the pool holding metadata information is called ceph-filesystem-metadata
- the Acting Set for PG1 is OSD1, OSD2 and OSD3. OSD3 is the primary and the client is contacts it directly.



## Cluster Map

So - if you have understood the above then you are ready to understand the idea of a cluster map

A collection of 5 maps
	monitor map 
	OSD map 
	placement group map 
	MDS (i think file system map)
	CRUSH map						-	contains list of storage devices, failure domain hierarchy (device, host, rack, row, room)
										rules for traversing the hierarchy



## Applications
Applications are things that use pools of storage

cephfs			-	pools that are intended for use with cephfs are associated automtically
rbd				-	pools intended for use with rbd should be initialized
rgw				-	pools created automatically by rados gateway are assocated automatically


this one is enabled automatically (i think)
mgr				-	the manager application has access to its own pool to store its information



## Profile
mds



## FSID
The identifier for the cluster



## Cephx
The ceph authentication protocol that authenticates clients and services


Terms

monitor		-	central authorization authority
service		-	the set of all daemons of a particular type (e.g. all OSDs, all MDSs)
client		-	an entity or principal that is accessing the service


# DAEMON and CLIENT ACCESS TO THE CLUSTER

## First let's think about who has access to the ceph cluster

```shell
k rook-ceph ceph auth ls | grep -v '^[[:blank:]]'

mon.
mds.ceph-filesystem-a
mds.ceph-filesystem-b
osd.0
osd.1
osd.10
osd.11
osd.12
osd.13
osd.14
osd.15
osd.16
osd.17
osd.18
osd.19
osd.2
osd.20
osd.21
osd.22
osd.23
osd.24
osd.25
osd.26
osd.3
osd.4
osd.5
osd.6
osd.7
osd.8
osd.9
client.admin
client.ceph-exporter
client.crash
client.csi-cephfs-node.2
client.csi-cephfs-provisioner.2
client.csi-rbd-node.2
client.csi-rbd-provisioner.2
client.osd-lockbox.07ac1607-d37a-4cf9-88a1-5d29b8534ed6
client.osd-lockbox.0f68f171-a63e-4cd2-a40c-4700c57ec724
client.osd-lockbox.225ad83d-d9d5-46fe-a460-1624bfb23350
client.osd-lockbox.23b1c8d5-1b4a-40ab-a0a0-206d40ea7a4c
client.osd-lockbox.24b0a12e-5eb8-4669-aaf4-d5e62d170795
client.osd-lockbox.2c76e8f2-71c0-461d-be66-8d5238b2e492
...
...
client.rbd-mirror-peer
client.rgw.ceph.objectstore.a
mgr.a
mgr.b
```

There are clients 		- these will start with client
There are services 		- these will start with mgr, mon, osd, mds


There is a concept called 'rotating services'
These services keep rotating their shared secret keys
instead of static permanent secrets
the 4 are - mon, mds, osd and mgr

## Accessing CephFS

the client can use either
- a kernel driver (kcephfs, ceph.ko, libceph)
- a userspace utility (ceph-fuse)

auth with cephx
access monitor to get map
access mds to get metadata
access OSD for read/write


## Accessing RBD


## Accessing Rados Gateway



# CEPHFS

In CephFS, 
- a CephFS volume is the top-level abstraction for an entire CephFS file system, 
- while a subvolume group is a container for multiple subvolumes that allows you to apply common policies, such as file layouts or snapshots, across them. 
- Subvolumes themselves are independent CephFS directory trees that function as individual file system units. 

NOTE: an instance of cephfs is represented by a ```ceph fs volume ls```


## CephFS volume

It seems that a ceph filesystem is considered a 'volume' in ceph (or think of it as an FS volume is an abstraction of CephFS file system). 
The ceph filesystems that have been provisioned are available with the command 'ceph fs volume ls'


## CephFS Subvolume Group


## CephFS subvolumes





# RBD

## Rados Namespace
a segregation in a pool to allow for better management
only available when using librados


# CEPH-CSI

https://docs.ceph.com/en/latest/csi/

## CEPHFS

### csidriver

cm/ceph-config                                          # think about this as how the ceph cluster is configured
cm/ceph-csi-config                                      # think about this as how ceph-csi should be configured to talk to the ceph cluster, each cluster is an array item
                                                        # the clusterID + monitors, the rbd, cephfs, nfs settings, the readAffinity setting 
cm/ceph-csi-encryption-kms-config

#### csi-cephfs-plugin-provisioner
sa, cr, crb, role, rb for provisioner                   # rbac for provisioner
deployment/csi-cephfs-plugin-provisioner                # i believe this provisioner is also a 'controller' and is watching for pvc requests to the api

  containers
        csi-cephfsplugin 
        csi-provisioner                                 # listening for events that create pvc
        csi-resizer
        csi-snapshotter
        liveness-prometheus


#### csi-cephfs-plugin-provisioner
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



### sc/cephfs.csi.ceph.com

The storage class will
 
    1. have a reclaim policy, allowvolumeexpansion and have a volumebinding mode

    2. know the filesystem and pool to use
    
    3. know the provisioner to use                  -   how is the provsioner determined? does the namespace have to be used in calling the provisioner

    4. know the cluster id

    5. provisioner-secret                           -   adminID and adminkey  (it invokes teh provisioner with this secret to talk to the storage cluster)

    6. node-stage-secret                            -   i think this is to mount the volume into the node and make available to the app

    7. controller-expand-secret                     -   what is this for?

    8. fstype


### Mounting 

Mounting of a cephfs volume to a node can happen via 

kernel 

fuse 		-	a way for non-priv users to create a filesystem without editing kernel code 





## CEPH-RBD

# COMMANDS

```shell
# HEALTH
ceph status | -s
ceph health detail 

# CONFIGURATION
ceph config dump
ceph config show <daemon>.<id>

# STORAGE
# show the total storage
ceph df

# HOST INFO
# host information for hosts in the cluster
ceph orch host ls --detail

# DAEMONS
# show a summary of the running daemons
ceph orch ls										

# show detailed of running daemons
ceph orch ps

# monitor dump
ceph mon dump

# mgr dump
ceph mgr dump

# osd dump
ceph osd dump													

# AUTH
ceph auth ls
ceph auth get client.<id>
ceph auth get-or-create client.<client_name> mon ‘profile rbd’ osd ‘profile rbd pool=<pool_name>, profile rbd-read-only pool=<pool_name>’ mgr ‘profile rbd pool=<pool_name>’



# OSD
# disks with weight and size
ceph osd df														

# see the layout of the storage according to OSD
ceph osd tree 

# show device information - the host for the device, the daemons on the device, etc.
ceph device ls

ceph osd crush rule create-replicated hddpool <root> <failure-domain> <disk-type>


# POOLS
# list the pools that OSD knows about
ceph osd lspools

ceph osd pool ls detail

# get all the settings for a pool (or get an individual setting, e.g. ceph osd pool get pool1 pg_num)
ceph osd pool get <pool_name> all

# e.g. ceph osd pool application enable liverpool rbd
ceph osd pool application enable <pool> <app>					



# RADOSGW
radosgw-admin user <option>
radosgw-admin user stats --uid=<uid>
radosgw-admin bucket <option>
# the default scope for quota is users, this will make ceph honour the quota on a bucket
radosgw-admin quota enable --quota-scope=bucket --uid=<uid>
radosgw-admin --show-config


# CEPHFS
# shows the mds map
ceph fs dump
# shows the ceph filesystems that have been configured on the system (ceph filesystems are perceived as 'volumes')
ceph fs volume ls	
ceph fs volume info <volume> [--human_readable]

ceph fs subvolumegroup ls <volume>
ceph fs subvolumegroup info <volume> <subvolumegroup>

ceph fs subvolume ls <volume> <subvolumegroup>
ceph fs subvolume info <volume> <subvolume> <subvolumegroup>


# RBD
# the pool is created by the 'ceph'command
rbd pool init <pool>
# the list of rbd images
rbd ls [<pool>]
rbd info <pool>/<image>

rbd namespace create <pool>/<namespace>
rbd namespace list <pool> 

# show the devices (rados block devices) across all the pools
rbd device list
```

WHAT IS THE DIFFERENCE BETWEEN IMAGE AND DEVICE? Not all images seem to be devices


# TROUBLESHOOTING

Placement Group states/status

             6149 active+clean
             17   active+clean+scrubbing
             16   active+remapped+backfill_wait
             14   active+clean+scrubbing+deep
             5    activating+degraded+remapped
             2    active+remapped+backfilling
             1    activating+remapped



$ k get cephclusters.ceph.rook.io -n rook-ceph -o yaml opsk8s | yq -r .spec.disruptionManagement
managePodBudgets: true
osdMaintenanceTimeout: 30



ACTUAL ISSUE of pgs: x/y objects misplaced

```shell
#find pg's that are stuck

$ k rook-ceph ceph pg dump_stuck
Info: running 'ceph' command with args: [pg dump_stuck]
ok
PG_STAT  STATE                          UP          UP_PRIMARY  ACTING      ACTING_PRIMARY
6.b9     active+remapped+backfill_wait  [61,38,58]          61  [38,58,47]              38
14.b8      active+remapped+backfilling  [61,65,64]          61  [15,65,64]              15
14.aa    active+remapped+backfill_wait  [68,38,57]          68  [68,38,15]              68
14.93    active+remapped+backfill_wait  [74,57,64]          74  [74,64,69]              74
14.f2    active+remapped+backfill_wait  [65,60,69]          65  [65,60,61]              65
14.88      active+remapped+backfilling  [48,69,58]          48  [48,58,57]              48
6.163    active+remapped+backfill_wait  [27,61,77]          27  [27,77,47]              27
14.100   active+remapped+backfill_wait  [69,68,27]          69  [68,27,57]              68
14.18d   active+remapped+backfill_wait   [63,75,5]          63   [75,5,57]              75
6.191    active+remapped+backfill_wait  [69,38,71]          69  [38,71,63]              38
6.73     active+remapped+backfill_wait  [69,74,67]          69   [74,67,3]              74
14.92    active+remapped+backfill_wait   [68,4,61]          68   [68,4,69]              68
6.1c5    active+remapped+backfill_wait  [16,39,61]          16  [16,39,47]              16
14.1ee   active+remapped+backfill_wait  [17,61,75]          17  [17,75,63]              17
6.188    active+remapped+backfill_wait  [71,69,56]          71  [71,56,63]              71
14.1c2   active+remapped+backfill_wait  [57,75,17]          57  [17,75,63]              17
14.5d    active+remapped+backfill_wait  [77,63,48]          77  [77,48,61]              77
14.1ed   active+remapped+backfill_wait  [38,57,58]          38  [38,58,63]              38
6.1d5    active+remapped+backfill_wait  [62,64,61]          62  [62,64,47]              62
14.14c     active+remapped+backfilling  [28,75,57]          28  [28,75,63]              28
14.a1    active+remapped+backfill_wait  [58,27,61]          58  [58,27,63]              58
14.12b   active+remapped+backfill_wait  [15,58,75]          15  [58,75,72]              58
14.19d   active+remapped+backfill_wait  [15,49,27]          15  [25,49,27]              25
14.82    active+remapped+backfill_wait  [56,17,63]          56  [56,17,61]              56
14.1e0   active+remapped+backfill_wait  [38,57,28]          38  [38,28,63]              38
14.10c   active+remapped+backfill_wait  [56,25,58]          56  [56,58,15]              56
6.1e9    active+remapped+backfill_wait  [17,63,64]          17  [17,64,47]              17
14.134   active+remapped+backfill_wait  [15,17,16]          15  [17,16,57]              17
14.a4    active+remapped+backfill_wait  [71,57,75]          71  [71,75,61]              71
6.69     active+remapped+backfill_wait  [62,63,38]          62  [62,38,47]              62
14.10b   active+remapped+backfill_wait  [68,57,38]          68  [68,38,69]              68
14.1c1     active+remapped+backfilling  [39,70,63]          39  [39,70,57]              39
14.63    active+remapped+backfill_wait   [5,60,63]           5   [5,60,57]               5
14.fd    active+remapped+backfill_wait  [63,73,39]          63  [73,39,61]              73
14.4c    active+remapped+backfill_wait  [27,63,39]          27  [27,39,57]              27
14.72    active+remapped+backfill_wait  [61,71,64]          61  [57,71,64]              57
14.1a      active+remapped+backfilling  [65,38,72]          65  [65,38,15]              65
14.2     active+remapped+backfill_wait  [57,75,62]          57  [62,75,63]              62
6.86     active+remapped+backfill_wait   [63,4,74]          63   [4,74,37]               4
14.71      active+remapped+backfilling  [16,74,15]          16  [16,74,57]              16




# query PG to find backfill targets
$ k rook-ceph ceph pg 6.86 query
Info: running 'ceph' command with args: [pg 6.86 query]
{
    "snap_trimq": "[]",
    "snap_trimq_len": 0,
    "state": "active+remapped+backfill_wait",
    "epoch": 37189,
    "up": [
        63,
        4,
        74
    ],
    "acting": [
        4,
        74,
        37
    ],
    "backfill_targets": [
        "63"
    ],
    "acting_recovery_backfill": [
        "4",
        "37",
        "63",
        "74"
    ],
    "info": {
        "pgid": "6.86",
        "shared": "255",
        "last_update": "1127'14644",
        "last_complete": "1127'14644",
        "log_tail": "1127'13200",
        "last_user_version": 14644,
        "last_backfill": "MAX",
        "purged_snaps": [],
        "history": {
            "epoch_created": 990,
            "epoch_pool_created": 980,
            "last_epoch_started": 37171,
            "last_interval_started": 37170,
            "last_epoch_clean": 36988,
            "last_interval_clean": 36987,
            "last_epoch_split": 998,
            "last_epoch_marked_full": 0,
            "same_up_since": 37169,
            "same_interval_since": 37170,
            "same_primary_since": 37170,
            "last_scrub": "1127'14644",
            "last_scrub_stamp": "2026-10-22T01:19:05.534540+0000",
            "last_deep_scrub": "1127'14644",
            "last_deep_scrub_stamp": "2026-10-20T16:42:42.090242+0000",
            "last_clean_scrub_stamp": "2026-10-22T01:19:05.534540+0000",
            "prior_readable_until_ub": 0
        },
        "stats": {
            "version": "1127'14644",
            "reported_seq": 167052,
            "reported_epoch": 37189,
            "state": "active+remapped+backfill_wait",
            "last_fresh": "2026-10-22T03:12:41.104120+0000",
            "last_change": "2026-10-22T02:59:10.045394+0000",
            "last_active": "2026-10-22T03:12:41.104120+0000",
            "last_peered": "2026-10-22T03:12:41.104120+0000",
            "last_clean": "2025-05-28T02:39:17.814309+0000",
            "last_became_active": "2026-10-22T02:59:10.045277+0000",
            "last_became_peered": "2026-10-22T02:59:10.045277+0000",
            "last_unstale": "2026-10-22T03:12:41.104120+0000",
            "last_undegraded": "2026-10-22T03:12:41.104120+0000",
            "last_fullsized": "2026-10-22T03:12:41.104120+0000",
            "mapping_epoch": 37170,
            "log_start": "1127'13200",
            "ondisk_log_start": "1127'13200",
            "created": 990,
            "last_epoch_clean": 36988,
            "parent": "0.0",
            "parent_split_bits": 8,
            "last_scrub": "1127'14644",
            "last_scrub_stamp": "2026-10-22T01:19:05.534540+0000",
            "last_deep_scrub": "1127'14644",
            "last_deep_scrub_stamp": "2026-10-20T16:42:42.090242+0000",
            "last_clean_scrub_stamp": "2026-10-22T01:19:05.534540+0000",
            "objects_scrubbed": 0,
            "log_size": 1444,
            "log_dups_size": 3000,
            "ondisk_log_size": 1444,
            "stats_invalid": false,
            "dirty_stats_invalid": false,
            "omap_stats_invalid": false,
            "hitset_stats_invalid": false,
            "hitset_bytes_stats_invalid": false,
            "pin_stats_invalid": false,
            "manifest_stats_invalid": false,
            "snaptrimq_len": 0,
            "last_scrub_duration": 1,
            "scrub_schedule": "no scrub is scheduled",
            "scrub_duration": 772,
            "objects_trimmed": 0,
            "snaptrim_duration": 0,
            "stat_sum": {
                "num_bytes": 0,
                "num_objects": 0,
                "num_object_clones": 0,
                "num_object_copies": 0,
                "num_objects_missing_on_primary": 0,
                "num_objects_missing": 0,
                "num_objects_degraded": 0,
                "num_objects_misplaced": 0,
                "num_objects_unfound": 0,
                "num_objects_dirty": 0,
                "num_whiteouts": 0,
                "num_read": 151295,
                "num_read_kb": 9669932,
                "num_write": 14634,
                "num_write_kb": 3379320,
                "num_scrub_errors": 0,
                "num_shallow_scrub_errors": 0,
                "num_deep_scrub_errors": 0,
                "num_objects_recovered": 0,
                "num_bytes_recovered": 0,
                "num_keys_recovered": 0,
                "num_objects_omap": 0,
                "num_objects_hit_set_archive": 0,
                "num_bytes_hit_set_archive": 0,
                "num_flush": 0,
                "num_flush_kb": 0,
                "num_evict": 0,
                "num_evict_kb": 0,
                "num_promote": 0,
                "num_flush_mode_high": 0,
                "num_flush_mode_low": 0,
                "num_evict_mode_some": 0,
                "num_evict_mode_full": 0,
                "num_objects_pinned": 0,
                "num_legacy_snapsets": 0,
                "num_large_omap_objects": 0,
                "num_objects_manifest": 0,
                "num_omap_bytes": 0,
                "num_omap_keys": 0,
                "num_objects_repaired": 0
            },
            "up": [
                63,
                4,
                74
            ],
            "acting": [
                4,
                74,
                37
            ],
            "avail_no_missing": [
                "4",
                "37",
                "63",
                "74"
            ],
            "object_location_counts": [],
            "blocked_by": [],
            "up_primary": 63,
            "acting_primary": 4,
            "purged_snaps": []
        },
        "empty": 0,
        "dne": 0,
        "incomplete": 0,
        "last_epoch_started": 37171,
        "hit_set_history": {
            "current_last_update": "0'0",
            "history": []
        }
    },
    "peer_info": [
        {
            "peer": "37",
            "pgid": "6.86",
            "shared": "255",
            "last_update": "1127'14644",
            "last_complete": "1127'14644",
            "log_tail": "1127'13200",
            "last_user_version": 14644,
            "last_backfill": "MAX",
            "purged_snaps": [],
            "history": {
                "epoch_created": 990,
                "epoch_pool_created": 980,
                "last_epoch_started": 37171,
                "last_interval_started": 37170,
                "last_epoch_clean": 36988,
                "last_interval_clean": 36987,
                "last_epoch_split": 998,
                "last_epoch_marked_full": 0,
                "same_up_since": 37169,
                "same_interval_since": 37170,
                "same_primary_since": 37170,
                "last_scrub": "1127'14644",
                "last_scrub_stamp": "2026-10-22T01:19:05.534540+0000",
                "last_deep_scrub": "1127'14644",
                "last_deep_scrub_stamp": "2026-10-20T16:42:42.090242+0000",
                "last_clean_scrub_stamp": "2026-10-22T01:19:05.534540+0000",
                "prior_readable_until_ub": 0
            },
            "stats": {
                "version": "1127'14644",
                "reported_seq": 567504,
                "reported_epoch": 37168,
                "state": "active+clean",
                "last_fresh": "2026-10-22T02:58:29.890625+0000",
                "last_change": "2026-10-22T02:27:05.826953+0000",
                "last_active": "2026-10-22T02:58:29.890625+0000",
                "last_peered": "2026-10-22T02:58:29.890625+0000",
                "last_clean": "2026-10-22T02:58:29.890625+0000",
                "last_became_active": "2026-10-22T02:27:05.826827+0000",
                "last_became_peered": "2026-10-22T02:27:05.826827+0000",
                "last_unstale": "2026-10-22T02:58:29.890625+0000",
                "last_undegraded": "2026-10-22T02:58:29.890625+0000",
                "last_fullsized": "2026-10-22T02:58:29.890625+0000",
                "mapping_epoch": 37170,
                "log_start": "1127'13200",
                "ondisk_log_start": "1127'13200",
                "created": 990,
                "last_epoch_clean": 36988,
                "parent": "0.0",
                "parent_split_bits": 8,
                "last_scrub": "1127'14644",
                "last_scrub_stamp": "2026-10-22T01:19:05.534540+0000",
                "last_deep_scrub": "1127'14644",
                "last_deep_scrub_stamp": "2026-10-20T16:42:42.090242+0000",
                "last_clean_scrub_stamp": "2026-10-22T01:19:05.534540+0000",
                "objects_scrubbed": 0,
                "log_size": 1444,
                "log_dups_size": 3000,
                "ondisk_log_size": 1444,
                "stats_invalid": false,
                "dirty_stats_invalid": false,
                "omap_stats_invalid": false,
                "hitset_stats_invalid": false,
                "hitset_bytes_stats_invalid": false,
                "pin_stats_invalid": false,
                "manifest_stats_invalid": false,
                "snaptrimq_len": 0,
                "last_scrub_duration": 1,
                "scrub_schedule": "periodic scrub scheduled @ 2026-10-23T11:35:07.718763+0000",
                "scrub_duration": 772,
                "objects_trimmed": 0,
                "snaptrim_duration": 0,
                "stat_sum": {
                    "num_bytes": 0,
                    "num_objects": 0,
                    "num_object_clones": 0,
                    "num_object_copies": 0,
                    "num_objects_missing_on_primary": 0,
                    "num_objects_missing": 0,
                    "num_objects_degraded": 0,
                    "num_objects_misplaced": 0,
                    "num_objects_unfound": 0,
                    "num_objects_dirty": 0,
                    "num_whiteouts": 0,
                    "num_read": 151295,
                    "num_read_kb": 9669932,
                    "num_write": 14634,
                    "num_write_kb": 3379320,
                    "num_scrub_errors": 0,
                    "num_shallow_scrub_errors": 0,
                    "num_deep_scrub_errors": 0,
                    "num_objects_recovered": 0,
                    "num_bytes_recovered": 0,
                    "num_keys_recovered": 0,
                    "num_objects_omap": 0,
                    "num_objects_hit_set_archive": 0,
                    "num_bytes_hit_set_archive": 0,
                    "num_flush": 0,
                    "num_flush_kb": 0,
                    "num_evict": 0,
                    "num_evict_kb": 0,
                    "num_promote": 0,
                    "num_flush_mode_high": 0,
                    "num_flush_mode_low": 0,
                    "num_evict_mode_some": 0,
                    "num_evict_mode_full": 0,
                    "num_objects_pinned": 0,
                    "num_legacy_snapsets": 0,
                    "num_large_omap_objects": 0,
                    "num_objects_manifest": 0,
                    "num_omap_bytes": 0,
                    "num_omap_keys": 0,
                    "num_objects_repaired": 0
                },
                "up": [
                    63,
                    4,
                    74
                ],
                "acting": [
                    4,
                    74,
                    37
                ],
                "avail_no_missing": [],
                "object_location_counts": [],
                "blocked_by": [],
                "up_primary": 63,
                "acting_primary": 4,
                "purged_snaps": []
            },
            "empty": 0,
            "dne": 0,
            "incomplete": 0,
            "last_epoch_started": 37171,
            "hit_set_history": {
                "current_last_update": "0'0",
                "history": []
            }
        },
        {
            "peer": "63",
            "pgid": "6.86",
            "shared": "255",
            "last_update": "1127'14644",
            "last_complete": "1127'14644",
            "log_tail": "1127'13200",
            "last_user_version": 0,
            "last_backfill": "MIN",
            "purged_snaps": [],
            "history": {
                "epoch_created": 990,
                "epoch_pool_created": 980,
                "last_epoch_started": 37171,
                "last_interval_started": 37170,
                "last_epoch_clean": 36988,
                "last_interval_clean": 36987,
                "last_epoch_split": 998,
                "last_epoch_marked_full": 0,
                "same_up_since": 37169,
                "same_interval_since": 37170,
                "same_primary_since": 37170,
                "last_scrub": "1127'14644",
                "last_scrub_stamp": "2026-10-22T01:19:05.534540+0000",
                "last_deep_scrub": "1127'14644",
                "last_deep_scrub_stamp": "2026-10-20T16:42:42.090242+0000",
                "last_clean_scrub_stamp": "2026-10-22T01:19:05.534540+0000",
                "prior_readable_until_ub": 0
            },
            "stats": {
                "version": "0'0",
                "reported_seq": 1,
                "reported_epoch": 37169,
                "state": "peering",
                "last_fresh": "2026-10-22T02:59:08.001308+0000",
                "last_change": "2026-10-22T02:59:08.001308+0000",
                "last_active": "0.000000",
                "last_peered": "0.000000",
                "last_clean": "0.000000",
                "last_became_active": "0.000000",
                "last_became_peered": "0.000000",
                "last_unstale": "2026-10-22T02:59:08.001308+0000",
                "last_undegraded": "2026-10-22T02:59:08.001308+0000",
                "last_fullsized": "2026-10-22T02:59:08.001308+0000",
                "mapping_epoch": 37170,
                "log_start": "0'0",
                "ondisk_log_start": "0'0",
                "created": 990,
                "last_epoch_clean": 36988,
                "parent": "0.0",
                "parent_split_bits": 0,
                "last_scrub": "1127'14644",
                "last_scrub_stamp": "2026-10-22T01:19:05.534540+0000",
                "last_deep_scrub": "1127'14644",
                "last_deep_scrub_stamp": "2026-10-20T16:42:42.090242+0000",
                "last_clean_scrub_stamp": "2026-10-22T01:19:05.534540+0000",
                "objects_scrubbed": 0,
                "log_size": 0,
                "log_dups_size": 0,
                "ondisk_log_size": 0,
                "stats_invalid": false,
                "dirty_stats_invalid": false,
                "omap_stats_invalid": false,
                "hitset_stats_invalid": false,
                "hitset_bytes_stats_invalid": false,
                "pin_stats_invalid": false,
                "manifest_stats_invalid": false,
                "snaptrimq_len": 0,
                "last_scrub_duration": 0,
                "scrub_schedule": "no scrub is scheduled",
                "scrub_duration": 0,
                "objects_trimmed": 0,
                "snaptrim_duration": 0,
                "stat_sum": {
                    "num_bytes": 0,
                    "num_objects": 0,
                    "num_object_clones": 0,
                    "num_object_copies": 0,
                    "num_objects_missing_on_primary": 0,
                    "num_objects_missing": 0,
                    "num_objects_degraded": 0,
                    "num_objects_misplaced": 0,
                    "num_objects_unfound": 0,
                    "num_objects_dirty": 0,
                    "num_whiteouts": 0,
                    "num_read": 0,
                    "num_read_kb": 0,
                    "num_write": 0,
                    "num_write_kb": 0,
                    "num_scrub_errors": 0,
                    "num_shallow_scrub_errors": 0,
                    "num_deep_scrub_errors": 0,
                    "num_objects_recovered": 0,
                    "num_bytes_recovered": 0,
                    "num_keys_recovered": 0,
                    "num_objects_omap": 0,
                    "num_objects_hit_set_archive": 0,
                    "num_bytes_hit_set_archive": 0,
                    "num_flush": 0,
                    "num_flush_kb": 0,
                    "num_evict": 0,
                    "num_evict_kb": 0,
                    "num_promote": 0,
                    "num_flush_mode_high": 0,
                    "num_flush_mode_low": 0,
                    "num_evict_mode_some": 0,
                    "num_evict_mode_full": 0,
                    "num_objects_pinned": 0,
                    "num_legacy_snapsets": 0,
                    "num_large_omap_objects": 0,
                    "num_objects_manifest": 0,
                    "num_omap_bytes": 0,
                    "num_omap_keys": 0,
                    "num_objects_repaired": 0
                },
                "up": [
                    63,
                    4,
                    74
                ],
                "acting": [
                    4,
                    74,
                    37
                ],
                "avail_no_missing": [],
                "object_location_counts": [],
                "blocked_by": [
                    4,
                    37,
                    74
                ],
                "up_primary": 63,
                "acting_primary": 4,
                "purged_snaps": []
            },
            "empty": 0,
            "dne": 0,
            "incomplete": 1,
            "last_epoch_started": 37171,
            "hit_set_history": {
                "current_last_update": "0'0",
                "history": []
            }
        },
        {
            "peer": "74",
            "pgid": "6.86",
            "shared": "255",
            "last_update": "1127'14644",
            "last_complete": "1127'14644",
            "log_tail": "1127'13200",
            "last_user_version": 14644,
            "last_backfill": "MAX",
            "purged_snaps": [],
            "history": {
                "epoch_created": 990,
                "epoch_pool_created": 980,
                "last_epoch_started": 37171,
                "last_interval_started": 37170,
                "last_epoch_clean": 36988,
                "last_interval_clean": 36987,
                "last_epoch_split": 998,
                "last_epoch_marked_full": 0,
                "same_up_since": 37169,
                "same_interval_since": 37170,
                "same_primary_since": 37170,
                "last_scrub": "1127'14644",
                "last_scrub_stamp": "2026-10-22T01:19:05.534540+0000",
                "last_deep_scrub": "1127'14644",
                "last_deep_scrub_stamp": "2026-10-20T16:42:42.090242+0000",
                "last_clean_scrub_stamp": "2026-10-22T01:19:05.534540+0000",
                "prior_readable_until_ub": 0
            },
            "stats": {
                "version": "1127'14643",
                "reported_seq": 166309,
                "reported_epoch": 1127,
                "state": "active+clean",
                "last_fresh": "2025-05-28T02:39:17.814309+0000",
                "last_change": "2025-05-28T00:55:50.116346+0000",
                "last_active": "2025-05-28T02:39:17.814309+0000",
                "last_peered": "2025-05-28T02:39:17.814309+0000",
                "last_clean": "2025-05-28T02:39:17.814309+0000",
                "last_became_active": "2025-05-28T00:55:32.809859+0000",
                "last_became_peered": "2025-05-28T00:55:32.809859+0000",
                "last_unstale": "2025-05-28T02:39:17.814309+0000",
                "last_undegraded": "2025-05-28T02:39:17.814309+0000",
                "last_fullsized": "2025-05-28T02:39:17.814309+0000",
                "mapping_epoch": 37170,
                "log_start": "1127'13200",
                "ondisk_log_start": "1127'13200",
                "created": 990,
                "last_epoch_clean": 999,
                "parent": "0.0",
                "parent_split_bits": 8,
                "last_scrub": "0'0",
                "last_scrub_stamp": "2025-05-28T00:55:50.116284+0000",
                "last_deep_scrub": "0'0",
                "last_deep_scrub_stamp": "2025-05-28T00:54:24.679242+0000",
                "last_clean_scrub_stamp": "2025-05-28T00:55:50.116284+0000",
                "objects_scrubbed": 0,
                "log_size": 1443,
                "log_dups_size": 3000,
                "ondisk_log_size": 1443,
                "stats_invalid": false,
                "dirty_stats_invalid": false,
                "omap_stats_invalid": false,
                "hitset_stats_invalid": false,
                "hitset_bytes_stats_invalid": false,
                "pin_stats_invalid": false,
                "manifest_stats_invalid": false,
                "snaptrimq_len": 0,
                "last_scrub_duration": 1,
                "scrub_schedule": "periodic scrub scheduled @ 2025-05-29T07:21:43.825509+0000",
                "scrub_duration": 772,
                "objects_trimmed": 0,
                "snaptrim_duration": 0,
                "stat_sum": {
                    "num_bytes": 0,
                    "num_objects": 0,
                    "num_object_clones": 0,
                    "num_object_copies": 3,
                    "num_objects_missing_on_primary": 0,
                    "num_objects_missing": 0,
                    "num_objects_degraded": 0,
                    "num_objects_misplaced": 0,
                    "num_objects_unfound": 0,
                    "num_objects_dirty": 0,
                    "num_whiteouts": 0,
                    "num_read": 151295,
                    "num_read_kb": 9669932,
                    "num_write": 14634,
                    "num_write_kb": 3379320,
                    "num_scrub_errors": 0,
                    "num_shallow_scrub_errors": 0,
                    "num_deep_scrub_errors": 0,
                    "num_objects_recovered": 0,
                    "num_bytes_recovered": 0,
                    "num_keys_recovered": 0,
                    "num_objects_omap": 0,
                    "num_objects_hit_set_archive": 0,
                    "num_bytes_hit_set_archive": 0,
                    "num_flush": 0,
                    "num_flush_kb": 0,
                    "num_evict": 0,
                    "num_evict_kb": 0,
                    "num_promote": 0,
                    "num_flush_mode_high": 0,
                    "num_flush_mode_low": 0,
                    "num_evict_mode_some": 0,
                    "num_evict_mode_full": 0,
                    "num_objects_pinned": 0,
                    "num_legacy_snapsets": 0,
                    "num_large_omap_objects": 0,
                    "num_objects_manifest": 0,
                    "num_omap_bytes": 0,
                    "num_omap_keys": 0,
                    "num_objects_repaired": 0
                },
                "up": [
                    63,
                    4,
                    74
                ],
                "acting": [
                    4,
                    74,
                    37
                ],
                "avail_no_missing": [],
                "object_location_counts": [],
                "blocked_by": [],
                "up_primary": 63,
                "acting_primary": 4,
                "purged_snaps": []
            },
            "empty": 0,
            "dne": 0,
            "incomplete": 0,
            "last_epoch_started": 37171,
            "hit_set_history": {
                "current_last_update": "0'0",
                "history": []
            }
        }
    ],
    "recovery_state": [
        {
            "name": "Started/Primary/Active",
            "enter_time": "2026-10-22T02:59:10.019055+0000",
            "might_have_unfound": [],
            "recovery_progress": {
                "backfill_targets": [
                    "63"
                ],
                "waiting_on_backfill": [],
                "last_backfill_started": "MIN",
                "backfill_info": {
                    "begin": "MIN",
                    "end": "MIN",
                    "objects": []
                },
                "peer_backfill_info": [],
                "backfills_in_flight": [],
                "recovering": [],
                "pg_backend": {
                    "pull_from_peer": [],
                    "pushing": []
                }
            }
        },
        {
            "name": "Started",
            "enter_time": "2026-10-22T02:59:09.013348+0000"
        }
    ],
    "scrubber": {
        "active": false,
        "must_scrub": false,
        "must_deep_scrub": false,
        "must_repair": false,
        "need_auto": false,
        "scrub_reg_stamp": "2026-10-23T08:54:21.627607+0000",
        "schedule": "no scrub is scheduled"
    },
    "agent_state": {}
}