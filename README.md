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
