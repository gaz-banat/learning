

REALLY COOL GUID

https://speakerdeck.com/redhatlivestreaming/red-hat-openshift-subscription-and-sizing-guide




IMPORTANT LOCATIONS ON A RH SYSTEM FOR RHSM

/etc/yum.repos.d/redhat.repo				-	the repo file managed by RHSM

/etc/pki/entitlement/XXXYYYZZZNNNN.pem			-	the systems ssl cert
/etc/pki/entitlement/XXXYYYZZZNNNN-key.pem		-	the systems ssl client key

/etc/rhsm						-	rhsm configuration directory
	ca/
	facts/
	pluginconf.d/
	syspurpose/
	logging.conf
	rhsm.conf
	rhsm.conf.rpmnew




SOCKET, CORE, vCPU


A CPU can be a Socket (1 Physical CPU, hardware based) or a vCPU (software based)

Physical CPU/Socket
	can have one or more cores (each core is a part containing the L1 cache and can work independently of other cores)
	if it supports hyper-threading (intel) or simultaneous multi-threading (AMD) then the number of cores is doubled
	each thread is considered a logical processor
vCPU
	this is a software implementation of a cpu in a virtual machine
	it can be implemented with 1 or more cores from the physical cpu, e.g. 1 vCPU could be built with 2 cores
	





PRODUCTS
	RH PRODUCT CATEGORIES

	Infrastructure Management
	Cloud Computing
	Storage
	Runtimes
	Integration and Automation
	Mobile


	Note: software product/s or software product set/s




	LIFECYCLES OF PRODUCTS

	Production
	Maintenance 1
	Maintenance 2
	Note: There is a cadence to maintenance releases, e.g. RHEL7 maintenance support cadence is 6 months

	Extended		-	support beyond the lifecycle of the product


	SUPPORT

	Level				-			self support, standard, premium
	Type				-			L1, L2, L3 




SUBSCRIPTION

		 --------- uses an entitlement and registers to ---------
		|							|
		|							V
		System						Subscription
		^							|
		|							|
		 ---------- attaches to ---------------------------------



Subscription
  Product		
    Name			-	Red Hat Developer Subscription for Individuals
    SKU (Product SKU)		-	RH00798
    Support Level
    Entitlement			-	16
    Capacity per subscription	-
    Content
	Content1		-	repo1a, repo1b, repo1c
	Content2		-	repo2a, repo2b, repo2c
  Subscription Number		-	14366812
  Contract Number 
  Name
  Quantity			-	1
  Start Date
  End Date
  Status			-	e.g. Active




Registered systems are entitled to 
	support services
	errata
	patches
	upgrades 
	
access to knowledgebase





PRODUCT

Name			Red Hat Enterprise Linux Server
Label			Red_Hat_Enterprise_Linux_Server
Number of Repos		9



REPOSITORY

Repo ID
Repo Name
Repo URL
Enabled

Packages




RED HAT INSIGHTS

https://console.redhat.com/insights/subscriptions/rhel#SIDs=&tags=





CONTENT DELIVERY NETWORK (CDN)


Content = Repository Name, Repository Label, Repository Type 



RHSM

A hosted service on the Red Hat Customer Portal
On-premise access through Red Hat Satellite
Using Subscription Manager from a Red Hat Enterprise Linux system

url 		- 	https://access.redhat.com/management						

Red Hat Subscription Management allows administrators to identify the relationship between their systems and the subscriptions used by those systems from two different perspectives:

All active subscriptions for an account and which systems are consuming them
All systems profiled within the inventory and which subscriptions they are consuming


How to get started on a rhel system:
subscription-manager register		-	register your system using your RHN account username and password
subscription-manager attach --auto	-	attach the subscription to your host

Now you can yum install




RED HAT SATELLITE

on premise subscription management application






WHAT IS

Activation key			- 	A key created by an organization administrator with pre-configured settings that can be applied to a subscription on a new system during registration
					preferred method of registering systems as clients of RHN proxy or RHN Satellite Server
					Systems registered using an activation key are subscribed to all channels associated with that activation key

Attach				-	Assigning a subscription to a host system for a Red Hat product, which consumes one or more entitlements based on the system architecture and the product being attached

Auto-attach			-	A feature in Red Hat Subscription Management and Subscription manager that allows Red Hat to scan for and automatically attach a subscription based on the system’s architecture

Entitlement			-	One of a pre-defined number of allowances a user has for registering their systems against a Red Hat Product

Errata				-	Updates that correct previously mistakes not caught before a software release

Organization Admin		-	Role that can manage users and control their access and permissions on an account

Subscription			-	A contract between Red Hat and a customer for a fixed period that provides access to content support, and the knowledge base

system facts			-

contract information		-

UUID				-





RED HAT SOFTWARE COLLECTIONS




TBS

Product + Quantity + Entitlement + Next Renewal

