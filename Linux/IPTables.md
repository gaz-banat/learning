
STRUCTURE

Table
	Chain
		Rule
		Rule
	Chain
		Rule
		Rule
		Rule
Table
	Chain
		Rule



INBUILT TABLES and CHAINS

Filter			- 	DEFAULT TABLE, if none other specified

	Input			- right before being handed to a local process

	Output		- right after being created by a local process

	Forward		- for packets coming in one interface and leaving from another
	

Nat			- 	rewrite source or destination and/or track connections

	Prerouting		- immediately after being received by an interface

	Postrouting	- right before leaving an interface

	Output		- right after being created by a local process


Mangle		- 	otherwise modify packets, e.g. modify portions of a TCP header 

	Prerouting

	Postrouting

	Input

	Output

	Forward

Raw


Security





FLOW (with Mangle)

Destination is localhost		-	Interface — Mangle/Prerouting — Nat/Prerouting — Routing decision — Mangle/Input — Filter/Input — Local Process

Source is localhost			-	Local Process — Routing Decision — Mangle/Output — Nat/Output — Filter/Output — Mangle/Postrouting — Nat/Postrouting — Interface

Forwarded packets			-	Interface — Mangle/Prerouting — Nat/Prerouting — Routing Decsion — Mangle/Forward — Filter/Forward — Mangle/Postrouting — Nat/Postrouting — Interface



FLOW DESCRIPTION (with Mangle)

—Coming In from network—
Packets coming in from network go through PREROUTING (Mangle, NAT)
Make a routing decision
	Decision is localhost - go through INPUT (Mangle, Filter) and hand to local process
	Decision is to forward - go through FORWARD (Mangle, Filter) then POSTROUTING (Mangle, NAT) and send it out the interface

—from Local Process—
take a routing decision then go through OUTPUT (Mangle, NAT, Filter) then POSTROUTING (Mangle, NAT) and send it out the interface 




FLOW DESCRIPTION (without Mangle)

Coming in from network for Localhost 	—	NAT/PREROUTING and then FILTER/INPUT and then local process
Coming in from network to be forwarded 	—	NAT/PREROUTING and then FILTER/FORWARD  and then NAT/POSTROUTING and send it out the interface
Coming from Local Process			—	NAT/OUTPUT and then FILTER/OUTPUT and then NAT/POSTROUTING and send it out the interface




TARGETS

Accept		-		all good
DNAT		-		rewrite the destination IP address, only valid in Nat/Prerouting and Nat/Output
Drop			-		it like its hot
Log			-		uses the kernel logging facility (syslog, rsyslog, etc)
Mark			-		only valid in Mangle table
Masquerade	-		rewrite the source IP address, only valid in Nat/Postrouting
Queue
Redirect
Reject		-		like Drop but let the sender know
Return		-		stop evaluation of this chain and either return to parent chain or go to policy for this chain
SNAT		-		only valid in NAT/Postrouting
TOS
TTL			-		only valid in Mangle
ULOG



NOTES

The default action for a chain is called its policy
On forwarded packets no Input chain is touched
If a packet is Accepted in a rule on a chain then no more chains are processed in that table, however it could still get a drop on other rules in chains in a different table




COMMANDS

I want to see all the chains in the default table
iptables -L 

I want to see chains in a specific table
iptables -t <table> -L 

I want to see all chains in all tables
for table in filter nat mangle raw security; do echo == $table ==; iptables -t $table -L; echo -----------; done

I want to see rules in a specific chain
iptables -t <table>  -S <chain>

Get me the whole config of iptables
iptables-save




