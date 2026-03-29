

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

NOTE: rbd is a disk so good for small random IO workloads, cephfs is a filesystem so better suited for large sequential reads/writes and multi-user shared file environments.

## Daemons
Monitor (mon)						-	Maintain the cluster map and ensure consensus. (v2 is on port 3300, v1 is on port 6789)
Manager (mgr)						-	Provide additional monitoring and interface to external management systems
										state of the cluster = monitor map + OSD map + PG map + MDS map + CRUSH map
Object Storage Daemon (osd)			-	Store data on physical disks and handle replication, erasure coding, and recovery.
Meta Data Server (mds)				-	CephFS relies on Metadata Servers (MDS) for all metadata operations (file creation, deletion, lookups, attribute changes)
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

PlacementGroup status
active+clean
active+clean+scrubbing
active+clean+scrubbing+deep




## How it all hangs together


							Client (the client provides pool name, user and secret key)
							  |
							  |
							  |
							RADOS Object (client reads or writes a rados object)
							  |
MONITORS					  |		
			Monitor 1		Monitor 2		Monitor 3
							  |
							  |
							  |
  						      |
							  |
STORAGE INTERFACES			  |
	RBD						CephFS 						RADOSGW
							  |
							  |
							  |
							 MDS	  																I know the metadata server is involved in cephfs requests (but dont know where it sits)
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
OSD (Object Storage Daemon)	  |
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
	MDS 
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
The ceph authentication protocol that authenticates users and daemons




# CEPHFS

In CephFS, 
- a CephFS volume is the top-level abstraction for an entire CephFS file system, 
- while a subvolume group is a container for multiple subvolumes that allows you to apply common policies, such as file layouts or snapshots, across them. 
- Subvolumes themselves are independent CephFS directory trees that function as individual file system units. 

NOTE: an instance of cephfs is represented by a ceph fs volume


## CephFS volume

It seems that a ceph filesystem is considered a 'volume' in ceph (or think of it as an FS volume is an abstraction of CephFS file system). 
The ceph filesystems that have been provisioned are available with the command 'ceph fs volume ls'


## CephFS Subvolume Group


## CephFS subvolumes





# RBD

## Rados Namespace
a segregation in a pool to allow for better management
only available when using librados



# COMMANDS

ceph status | -s
ceph health detail 
ceph config dump
ceph config show <daemon>.<id>
ceph df												-	show the total storage

--

ceph orch ls										-	show a summary of the running daemons
ceph orch ps										-	show detailed of running daemons

ceph orch host ls --detail							-	host information for hosts in the cluster

--

ceph mon dump										-	shows the monitor map

--

ceph mgr dump

--

ceph osd dump													-	shows the osd map

ceph osd lspools												-	list the pools that OSD knows about
ceph osd pool ls detail
ceph osd pool get <pool_name> all		 						- 	get all the settings for a pool (or get an individual setting, e.g. ceph osd pool get pool1 pg_num)
ceph osd pool application enable <pool> <app>					- 	e.g. ceph osd pool application enable liverpool rbd

ceph osd df														-	disks with weight and size

ceph osd tree 													-	see the layout of the storage according to OSD


ceph osd crush rule create-replicated hddpool <root> <failure-domain> <disk-type>

--

ceph auth ls
ceph auth get client.<id>
ceph auth get-or-create client.<client_name> mon ‘profile rbd’ osd ‘profile rbd pool=<pool_name>, profile rbd-read-only pool=<pool_name>’ mgr ‘profile rbd pool=<pool_name>’


--

ceph device ls													-	show device information - the host for the device, the daemons on the device, etc.


--

radosgw-admin user <option>
radosgw-admin user stats --uid=<uid>
radosgw-admin bucket <option>
radosgw-admin quota enable --quota-scope=bucket --uid=<uid>		-	the default scope for quota is users, this will make ceph honour the quota on a bucket
radosgw-admin --show-config

--

ceph fs dump 													-	shows the mds map
ceph fs volume ls												-	shows the ceph filesystems that have been configured on the system (ceph filesystems are perceived as 'volumes')
ceph fs volume info <volume> [--human_readable]

ceph fs subvolumegroup ls <volume>
ceph fs subvolumegroup info <volume> <subvolumegroup>

ceph fs subvolume ls <volume> <subvolumegroup>
ceph fs subvolume info <volume> <subvolume> <subvolumegroup>

--

rbd pool init <pool>											-	the pool is created by the 'ceph'command
rbd ls [<pool>]													-	the list of rbd images
rbd info <pool>/<image>

rbd namespace create <pool>/<namespace>
rbd namespace list <pool> 

rbd device list													-	show the devices (rados block devices) across all the pools

WHAT IS THE DIFFERENCE BETWEEN IMAGE AND DEVICE? Not all images seem to be devices