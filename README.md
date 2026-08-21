# Dual-Stack 3-Tier-Hierarchy 

<img width="676" height="339" alt="Screenshot 2026-08-20 145555" src="https://github.com/user-attachments/assets/0d6d83d8-58f7-47b7-9998-efde2bd9c250" />

# Access Layer 
### Subnet Ranges / VLAN's

I started from the beginning with creating subnetworks (segmenting at layer-3) starting with the private Class B 172.16.0.0 /12. My first network for the VLAN 10 "PC's" needed an address range of 200 hosts so I chose the /24 CIDR notation which has 253 hosts in total. My second network is VLAN 20 "WAPs" and it needed 100 hosts. The next available network address is 172.16.1.0 and I used the /25 CIDR. VLAN 30 needed 50 hosts for "Server's" on the network 172.16.1.128. VLAN 40 needed 15 hosts for "Management" purposes and is also the native untagged VLAN on trunk ports. I used the 172.16.1.192 /27 with 30 hosts in total.

This is a Dual-Stack network which consists of IPV4 and IPV6 processes at once. I started the IPV6 networks with the prefix 2000:db8:1234:(VLAN number):: /64. I used the 4th hextet in the network prefix portion to represent the VLAN number of each subnet. I converted the binary of 10,20,30,40 into hexadecimal A,14,1E,28.

The purpose of the four VLAN's is to segment each physical switch (2960_1 and 2960_2) into 2 separate broadcast domains. Similar to how we segment at layer 3 for the reasoning of separate address ranges; and if there is a broadcast (255.255.255.255) in the destination ip address field, it will go to all logical ip addresses configured on end devices and intermediary devices in that given broadcast domain because every possible ip address fits within the broadcast, which in turn will lead each device to de-encapsulate the ethernet frame up the stack for itself. If multiple subnetworks connecting to a layer 2 switch had all their ports configured with VLAN 10, then an address range for one department of the building would be getting traffic from another department because the ethernet frames are freely transiting into the VLAN 10 port and flooding out all the other ports which includes different layer 3 networks. Having 1 subnet for 1 VLAN is generally rule of thumb. The switches logic is basic, looking at the all F's destination MAC and realizing that in binary that is all 1's matching all possible layer 2 addresses. an ARP broadcast message to a default gateway to know the address of its physical port, and a DHCP broadcast discover and request messages for finding the DHCP Server and receiving an ip address, are two examples of traffic using broadcasts in their respected layer 2 and/or layer 3 headers. 


In this example we see the layer 2 switch identifies that the G0/0 port is apart of the VLAN 10 broadcast domain, therefore if a PC sends a broadcast ethernet frame to the switchport, the switch will flood to only trunks or access ports that allow the VLAN 10 802.1q tagging, mitigating traffic congestion of 4 networks. 

<img width="245" height="331" alt="Screenshot 2026-08-20 193709" src="https://github.com/user-attachments/assets/d62bd6ed-4a34-4913-a753-2e92f28ec31c" />
<img width="370" height="202" alt="Screenshot 2026-08-20 193441" src="https://github.com/user-attachments/assets/d014b105-7bf8-4de7-a89b-1dfa26e7e283" />

### Etherchannel / Trunk Ports

Each access-layer switch (2960_1 and 2960_2) has two port-channels for the given interfaces. If the logical port-channel goes down or the physical cable itself on one group of ports, it has a redundant link going to the standby default gateway for the given VLAN. If a distribution MLS goes down (3560-x as an example), the access-layer switch still has a way to get to other networks/internet through the second MLS default gateway (3750-x). There are 5 channel groups in total in the topology all using the open standard LACP versus the closed Cisco PAGP. The duplex type, speed of each port, and allowed VLANs / Native VLAN must match in order to form an etherchannel. This was configured to load balance between the links in a group and to have an additional path to the distribution layer if a single pair is shutdown or fails. In the interface port-channel configurations I set the switchport to encapsulate dot1q, mode trunk, allowed VLAN of 10,20,30,40, and the native VLAN 40 for management traffic that is untagged without 802.1q field. 

In this example the access-layer switch (2960_1) receives an ethernet frame from an access port configured for VLAN 10 destined for the active VLAN 10 SVI on the distribution MLS (3560-x). It will look at its MAC address table at the mapping of the destination MAC / port ID, realizes the destination is through its port-channel group 2 interface, then encapsulates the 802.1q with the VLAN 10 ID information and sends it out of either G1/0 or G1/1 dependent on the load-balancing configurations of the ip header / ethernet header information.  

<img width="676" height="65" alt="Screenshot 2026-08-20 210404" src="https://github.com/user-attachments/assets/081b8c54-0cd2-4cd8-b7a6-0973bf8afd2c" />
<img width="431" height="154" alt="Screenshot 2026-08-20 210301" src="https://github.com/user-attachments/assets/35888e69-cd1e-43b9-998a-552c0b26bdc2" />











## Distribution Layer

## Core Layer
