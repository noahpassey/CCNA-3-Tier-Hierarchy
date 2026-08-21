# Dual-Stack 3-Tier-Hierarchy 

<img width="676" height="339" alt="Screenshot 2026-08-20 145555" src="https://github.com/user-attachments/assets/0d6d83d8-58f7-47b7-9998-efde2bd9c250" />

# Access Layer 

I started from the beginning with creating subnetworks (segmenting at layer-3) starting with the private Class B 172.16.0.0 /12. My first network for the VLAN 10 "PC's" needed an address range of 200 hosts so I chose the /24 CIDR notation which has 253 hosts in total. My second network is VLAN 20 "WAPs" and it needed 100 hosts. The next available network address is 172.16.1.0 and I used the /25 CIDR. VLAN 30 needed 50 hosts for "Server's" on the network 172.16.1.128. VLAN 40 needed 15 hosts for "Management" purposes and is also the native untagged VLAN on trunk ports. I used the 172.16.1.192 /27 with 30 hosts in total.

This is a Dual-Stack network which consists of IPV4 and IPV6 processes at once. I started the IPV6 networks with the prefix 2000:db8:1234:VLAN Number:: /64. I used the 4th hextet to represent the VLAN number of each subnet. I converted the binary of 10,20,30,40 into hexadecimal A,14,1E,28.

The purpose of the four VLAN's is to segment each physical switch (2960_1 and 2960_2) into 2 separate broadcast domains. Similar to how we segment at layer 3 for the reasoning of separate address ranges; and if there is a broadcast (255.255.255.255) in the destination ip address field, it will go to all logical ip addresses configured on end devices and intermediary devices in that given broadcast domain because every possible ip address fits within the broadcast, which in turn will lead each device to de-encapsulate the ethernet frame up the stack for itself. If multiple subnetworks connecting to a layer 2 switch had all their ports configured with VLAN 10, then an address range for one department of the building would be getting traffic from another department because the ethernet frames are freely transiting into the VLAN 10 port and flooding out all the other ports which includes different layer 3 networks. Having 1 subnet for 1 VLAN is generally rule of thumb. The switches logic is basic, looking at the all F's destination MAC and realizing that in binary that is all 1's matching all possible layer 2 addresses. an ARP broadcast message to a default gateway to know the address of its physical port, and a DHCP broadcast discover and request messages for finding the DHCP Server and receiving an ip address, are two examples of traffic using broadcasts in their respected layer 2 and/or layer 3 headers. 


In this example we see the layer 2 switch identifies that the G0/0 port is apart of the VLAN 10 broadcast domain, therefore if a PC as an example sends a broadcast ethernet frame to the switchport, the switch will flood to only trunks or access ports that allow the VLAN 10 802.1q tagging, mitigating traffic congestion of 4 networks. 

<img width="210" height="253" alt="Screenshot 2026-08-20 175608" src="https://github.com/user-attachments/assets/db2eefcd-e29e-4ca2-963b-3f4fea60e16a" />
<img width="370" height="202" alt="Screenshot 2026-08-20 193441" src="https://github.com/user-attachments/assets/d014b105-7bf8-4de7-a89b-1dfa26e7e283" />






## Distribution Layer

## Core Layer
