

TBC

The libnftnl library can be used for low-level interaction with nftables Netlink API over the libmnl library



FIRST UNDERSTAND NETFILTER



Flow for ip/ip6/inet families. Hooks are capitalized

										Local Process
										(listening on net_device)  Local Process
											|						|
											|						|
											|					Routing Decision
											|						|
											|						|
						    	    INPUT (filter chains)	OUTPUT (filter chains)
											|						|
											|						|
a network ------ net_device -------- PREROUTING  ------ Routing Decision---------- FORWARD ---------- POSTROUTING -------------------- net_device ------ a network
						            (nat chains)					            (filter chains)	      (nat chains)



THE FOUR NATS


SNAT				-		changes the source IP address, used for accessing the Internet :)

DNAT				-		changes the destination IP address, used for inbound traffic, port forward, 

Masquerade			-		changes the source IP address specifically to the outgoing interfaces IP address, so accessing the Internet from a home router

Redirect





CONNECTION TRACKING STATES

new
established
related
invalid




RULES 


can apply to:

ip 		-		ipv4 attributes
meta		-		metainformation about the packet like in interface name, out interface name
counter	-		run a counter for packets and bytes


decsions:

accept
drop
reject
jump		-	go to chain specified by jump
return	-	stop processing rules in this chain and continue at the next rule of the calling chain





LOCATIONS FOR NFTABLES

/etc/sysconfig/nftables.conf

/etc/nftables/



STRUCTURE

  Family

	Table

		Chain					Could be a base chain OR regular chain

			set
			set
			rule
			rule


		Chain


	Table

		Chain


  Family





FAMILIES

ip				-		for matching ipv4 packets
ip6				-		for matching ipv6 packets
inet				-		for both ip and ip6 families combined together
arp				-		for matching address resolution protocol (ARP) packets
bridge			-		for matching packets that traverse a bridge device
netdev			-		for matching packets from ingress





CHAINS

types:

  base				-		entry point for packets from the networking stack where a hook value is specified

	  filter			-		standard chain supported by ip, ip6, inet, bridge, arp families
							used for accepting or dropping packets

	  route			-		supported by ip, ip6 and inet families

	  nat				-		supported by ip, ip6 and inet families
							used for snat, dnat, masquerade and redirect functions
							only the first packet of a connection goes through this chain, with subsequent packets bypassing it, so can’t be used for filtering


  regular				-		used as a jump target for better organization




HOOKS


ip/ipv4/inet
	prerouting
	input
	forward
	output
	postrouting






HOW TO MAKE A RULE

Ask yourself what you want to do and where you want to hook into Netfilter

I want to filter packets going to a local process
use a basic chain of type filter and INPUT hook - now add your rules to that chain

I want to translate addresses befor they go out of the host
use a basic chain of type nat and POSTROUTING hook - now add your rules


I want to route packets from a local process to a destination using a specific gateway
use a basic chain of type route and POSTROUTING hook (TO BE CONFIRMED)




COMMANDS





