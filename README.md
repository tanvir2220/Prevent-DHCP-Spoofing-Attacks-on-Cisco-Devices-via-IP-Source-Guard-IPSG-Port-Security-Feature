<h1>Prevent DHCP Spoofing Attacks on Cisco Devices via IP Source Guard (IPSG) & Port Security Feature</h1>


<h2>Overview</h2>
The project involves configuration of two of the security features - IP Source Guard (IPSG) & Port Security- to prevent DHCP Spoofing attacks on Cisco devices.
<br />

<h2>Environments Used </h2>

- <b>Cisco Packet Tracer</b>


<h2>Program Walk-Through</h2>

IP Source Guard (IPSG)

•	IP Source Guard prevents IP and/or MAC address spoofing attacks on untrusted layer two interfaces

•	When IP source guard is enabled, all traffic is blocked except for DHCP packets

•	Once the host gets an IP address through DHCP, only the DHCP-assigned source IP address is permitted

•	You can also configure a static binding instead of using DHCP

•	Source guard is not a standalone tool

•	It relies on the information in the DHCP snooping database to do its work

•	You can only use this on layer two (access and trunk) interfaces and it only works inbound

Configuration

•	Let’s see how we can configure IP source guard

•	I’ll use the following topology

<p align="center">
<img src="https://i.ibb.co/4V3GFW6/1.png" height="80%" width="80%" alt="Prevent DHCP Spoofing Attacks on Cisco Devices via IP Source Guard (IPSG) & Port Security Feature"/>
<br />
<br />

•	H1 is a legitimate host that receives its IP address through DHCP

•	H2 is an attacker that tries to spoof its source IP address

•	S1 is a server with a static IP address

•	R1 assigns IP addresses through DHCP

•	SW1 is pre-configured with DHCP snooping

•	We will configure IP source guard on this switch

•	Let’s take a look at what we have

•	H1 receives an IP address through DHCP from R1

H1#show ip interface brief | include DHCP

FastEthernet0/0            192.168.1.1     YES DHCP   up                    up

•	We can see a binding in the DHCP snooping binding table

SW1#show ip source binding

MacAddress          	IpAddress        	Lease(sec)  	Type           	VLAN  	Interface

------------------  		---------------  	----------  	-------------  	----  	--------------------

00:1D:A1:8B:36:D0   	192.168.1.1      	86316       	dhcp-snooping   1     	GigabitEthernet0/1

Total number of bindings: 1

•	This information is important

•	We need it to make IP source guard work

DHCP Binding

•	Let’s configure IP Source guard

•	We’ll start with the interface that connects to H1

•	To enable this, you only need a single command:

SW1(config)#interface GigabitEthernet 0/1

SW1(config-if)#ip verify source

•	We can verify that it is enabled for the interface that connects to H1

SW1#show ip verify source

Interface  	Filter-type  	Filter-mode  	IP-address       	Mac-address     Vlan   	Log

---------  		-----------  	-----------  	---------------  	-----------------  	----   	---

Gi0/1      	ip           		active       	192.168.1.1                         	1      	disabled

•	SW1 now only permits source IP address 192.168.1.1 on the GigabitEthernet 0/1 interface

•	The MAC address field is empty so right now, the switch only checks the source IP address

•	We can also check the source MAC address though

•	IP source guard uses port-security for this

•	Here’s how to enable it

SW1(config)#interface GigabitEthernet 0/1

SW1(config-if)#switchport port-security

SW1(config-if)#ip verify source port-security

•	First, we enable port-security and then we add the port-security parameter to our ip verify source command

•	The MAC address now shows up in the table

SW1#show ip verify source

Interface  	Filter-type  	Filter-mode  	IP-address       	Mac-address              Vlan   Log

---------  		-----------  	-----------  	---------------  	-----------------  	         ----      ---

Gi0/1      	ip-mac       	active       	192.168.1.1      	00:1D:A1:8B:36:D0   1         disabled

•	SW1 now only permits the source IP address and source MAC address that we see in the table above

•	Let’s do a quick test

•	Let’s see if H1 can still ping R1

H1#ping 192.168.1.254

Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/4 ms

•	This is working, great

•	Let’s add the exact same commands on H2

SW1(config)#interface GigabitEthernet 0/2

SW1(config-if)#switchport port-security

SW1(config-if)#ip verify source port-security

•	H2 has a static IP address

•	Let’s check the table on SW1 again

SW1#show ip verify source 

Interface  	Filter-type  	Filter-mode  	IP-address       	Mac-address        	Vlan   	Log

---------  		-----------  	-----------  	---------------  	-----------------  		----   	---

Gi0/1      	ip-mac       	active       	192.168.1.1      00:1D:A1:8B:36:D0  	1      disabled

Gi0/2      	ip-mac       	active       	deny-all         	deny-all           		1

•	There is no known source IP and/or MAC address known on the GigabitEthernet 0/2 interface

•	So SW1 will drop everything

•	Let’s see if this is true, we can see it in action with a debug

SW1#debug ip verify source packet

Ip source guard debug packet debugging is on

•	Let’s send an IP packet from H2 to R1

H2#ping 192.168.1.254 repeat 1

Success rate is 0 percent (0/1)

•	This ping fails and SW1 will show us the following output

SW1#

DHCP_SECURITY_SW: validate port security packet, recv port: GigabitEthernet0/2, recv vlan: 1, mac: 0017.5aed.7af0, invalid flag: 1.

•	Great, this proves that IP source guard is working for us

Static Binding

•	What about that server?

•	It’s a legitimate device but it has a static IP address

•	Fortunately, we can create a Static Binding

•	Let’s check the MAC address of S1

S1#show interfaces FastEthernet 0/0 | include bia

Hardware is MV96340 Ethernet, address is 0016.c7be.0ec8 (bia 0016.c7be.0ec8)

•	We can use this to create a static binding for the IP address and MAC address of S1

•	Here’s how:

SW1(config)#ip source binding 0016.c7be.0ec8 vlan 1 192.168.1.200 interface GigabitEthernet 0/3

•	Let’s add the same commands we used for H1 and H2 on the interface that connects to S1

SW1(config)#interface GigabitEthernet 0/3

SW1(config-if)#switchport port-security

SW1(config-if)#ip verify source port-security

•	Now check the table on SW1

SW1#show ip verify source

Interface Filter-type  Filter-mode  IP-address          Mac-address         	Vlan   	Log

---------  	   -----------      -----------  	    ---------------         -----------------          	----   	---

Gi0/1       ip-mac        active       	   192.168.1.1        00:1D:A1:8B:36:D0  	1      	disabled

Gi0/2       ip-mac        active       	   deny-all               deny-all           	1

Gi0/3       ip-mac        active       	   192.168.1.200    00:16:C7:BE:0E:C8  	1      	disabled

•	This is looking good

•	We have a matching entry for the IP address of S1

•	Let’s try a quick ping

S1#ping 192.168.1.254

Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/4 ms

•	Excellent, this proves our static binding is working!

Conclusion

•	IP Source Guard is a security feature that restricts IP traffic on untrusted Layer 2 ports by filtering traffic based on:

o	DHCP Snooping Binding Database or

o	Manually Configured IP Source Bindings

•	This feature helps prevent IP spoofing attacks when a host tries to spoof and use the IP address of another host

•	Any IP traffic coming into the interface with a source IP address other than that assigned will be filtered out on the untrusted Layer 2 ports

# Port-Security on Cisco Switch

•	By default, there is no limit to the number of MAC addresses a switch can learn on an interface, and all MAC addresses are allowed

•	If we want, we can change this behaviour with Port Security

•	Let’s take a look at the following situation

<p align="center">
<img src="https://i.ibb.co/gPGRw0K/2.png" height="80%" width="80%" alt="Prevent DHCP Spoofing Attacks on Cisco Devices via IP Source Guard (IPSG) & Port Security Feature"/>
<br />
<br />

•	In the topology above, someone connected a cheap (unmanaged) switch that they brought from home to the FastEthernet 0/1 interface of our Cisco switch

•	Sometimes people like to bring an extra switch from home to the office

•	As a result, our Cisco switch will learn the MAC address of H1 and H2 on its FastEthernet 0/1 interface

•	Of course, we don’t want people to bring their own switches and connect them to our network

•	So, we want to prevent this from happening

•	This is how we can do it

Switch(config)#interface fa0/1

Switch(config-if)#switchport port-security

Switch(config-if)#switchport port-security maximum 1

•	Use the switchport port-security command to enable port security

•	I have configured port security, so only one MAC address is allowed

•	Once the switch sees another MAC address on the interface, it will be in Violation, and something will happen

•	I’ll show you what happens in a bit

•	Besides setting a maximum on the number of MAC addresses, we can also use port security to Filter MAC addresses

•	You can use this to only allow specific MAC addresses

•	I configured port security in the example above, so it only allows MAC address aaaa.bbbb.cccc

•	This is not the MAC address of my computer, so it’s perfect for demonstrating a violation

Switch(config)#interface fa0/1

Switch(config-if)#switchport port-security mac-address aaaa.bbbb.cccc

•	Use the switchport port-security mac-address command to define the MAC address that you want to allow

•	Now we’ll generate some traffic to cause a violation:

 C:\Documents and Settings\H1>ping 1.2.3.4
 
•	I’m pinging to some bogus IP address

•	There is nothing with IP address 1.2.3.4

•	I just want to generate some traffic

•	Here’s what you will see

 SwitchA#
 
%PM-4-ERR_DISABLE: psecure-violation error detected on Fa0/1, putting Fa0/1 in err-disable state

%PORT_SECURITY-2-PSECURE_VIOLATION: Security violation occurred, caused by MAC address 0090.cc0e.5023 on port FastEthernet0/1.

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to down

%LINK-3-UPDOWN: Interface FastEthernet0/1, changed state to down

•	We have a security violation, and as a result, the port goes in err-disable state

•	As you can see, it is now down

•	Let’s take a closer look at port security

Switch#show port-security interface fa0/1

Port Security              		: Enabled

Port Status                		: Secure-shutdown

Violation Mode             		: Shutdown

Aging Time                 		: 0 mins

Aging Type                 		: Absolute

SecureStatic Address Aging 	: Disabled

Maximum MAC Addresses      	: 1

Total MAC Addresses        	: 1

Configured MAC Addresses   	: 1

Sticky MAC Addresses       	: 0

Last Source Address:Vlan   	: 0090.cc0e.5023:1

Security Violation Count   	: 1

•	Here is a useful command to check your port security configuration

•	Use show port-security interface to see the port security details per interface

•	You can see the violation mode is shutdown and that the last violation was caused by MAC address 0090.cc0e.5023 (H1)

Switch#show interfaces fa0/1

FastEthernet0/1 is down, line protocol is down (err-disabled)

•	Shutting the interface after a security violation is a good idea (security-wise), but the problem is that the interface will stay in err-disable state

•	This probably means another call to the helpdesk and you bringing the interface back to the land of the living!

•	Let’s activate it again

Switch(config)#interface fa0/1

Switch(config-if)#shutdown

Switch(config-if)#no shutdown

•	To get the interface out of the err-disable state, you need to type shutdown followed by no shutdown

•	Only typing no shutdown is not enough

•	It might be easier if the interface could recover itself after a certain time

•	You can enable this with the following command

Switch(config)#errdisable recovery cause psecure-violation

•	After 5 minutes (300 seconds), it will automatically recover from the err-disable state

•	Make sure you solve the problem, though, because otherwise, it will just have another violation and end up in an err-disable state again

•	You can speed this up by changing the timer

•	Let’s set it to 30 seconds

SW1(config)#errdisable recovery interval 30

•	Instead of typing in the MAC address ourselves, we can also make the switch learn a MAC address for port security

Switch(config-if)#no switchport port-security mac-address aaaa.bbbb.cccc

Switch(config-if)#switchport port-security mac-address sticky

•	The sticky keyword will ensure that the switch uses the first MAC address that it learns on the interface for port security

•	Let’s verify it

Switch#show run interface fa0/1

Building configuration...

Current configuration : 228 bytes 

!

interface FastEthernet0/1

switchport mode access

switchport port-security

switchport port-security mac-address sticky

switchport port-security mac-address sticky 000c.2928.5c6c

•	You can see that it will save the MAC address of H1 in the running configuration by itself

•	Shutting the interface in case of a violation might be a bit too much

•	There are other options, here’s what you can do:

Switch(config-if)#switchport port-security violation ?

protect   	Security violation protect mode

restrict  	Security violation restrict mode

shutdown  	Security violation shutdown mode

•	There are other options like Protect and Restrict

o	Protect: Ethernet frames from MAC addresses that are not allowed will be dropped but you won’t receive any logging information

o	Restrict: Ethernet frames from MAC addresses that are not allowed will be dropped but you will see logging information and a SNMP trap is sent

o	Shutdown: Ethernet frames from MAC addresses that are not allowed will cause the interface to go to err-disable state.

You will see logging information and a SNMP trap is sent

•	For recovery you have two options

o	Manual: Recover the interface yourself with a shutdown and no shutdown

o	Automatic: use the errdisable recovery commands to enable and tune automatic recovery

