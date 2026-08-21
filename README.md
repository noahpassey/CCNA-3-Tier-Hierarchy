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

In this example the access-layer switch (2960_1) transits an ethernet frame from an an end-device. It has an access-port on the switch configured for VLAN 10, and the frame is destined for the active VLAN 10 SVI on the distribution MLS (3560-x). It will look inside the MAC address table for the mapping of the destination MAC / port ID, realizing the destination is through its port-channel group 2 interface, then encapsulates the 802.1q (VLAN 10 ID information), and sends it out of either G1/0 or G1/1 dependent on the load-balancing configurations of the ip header / ethernet header address information. This is a known-unicast frame. Unknown-unicast (meaning the destination is not in the MAC table) will be flooded out all ports except the port it was received on. Only the intended device will reply back (matching the binary 48 bits of the ethernet header address to its own address), which the source MAC of 3560-x replying will be exactly what we will need for future preference, as it will transit through the 2960_1 switch and it'll add this source MAC to its table and the port ID in which the frame entered. 

<img width="676" height="65" alt="Screenshot 2026-08-20 210404" src="https://github.com/user-attachments/assets/081b8c54-0cd2-4cd8-b7a6-0973bf8afd2c" />
<img width="431" height="154" alt="Screenshot 2026-08-20 210301" src="https://github.com/user-attachments/assets/35888e69-cd1e-43b9-998a-552c0b26bdc2" />
<img width="492" height="119" alt="Screenshot 2026-08-20 214828" src="https://github.com/user-attachments/assets/9f249391-80e1-41c9-a78a-5b368fa65664" />

### PVSTP

I wanted each VLAN to have its own STP instance, meaning different port types / root bridge per VLAN. This protocol has specific rules in order to elect the root bridge, root ports, designated ports and blocking ports. As well as port states. When you connect a switch to power, all ports start in the blocking state. In this state the port cannot send BPDUs but can receive BPDUs, it does not learn MAC addresses and it doesn't send or receive regular data traffic (ethernet frames). All switches start by assuming it is the root bridge and it will only give up its position if it receives superior BPDU's (lower BID). The next state is the Listening state. This is a transitional state that lasts 15 seconds by default which is determined by the Forward Delay Timer to ensure there are no loops within the topology. This is the switch starts sending its own BPDUs to other neighbor switches, while not sending or receiving regular traffic and not learning MAC addresses. The Learning State also has a 15 second timer and this is also a transitional state for only ports moving into a designated or root port role because of a topology change. The main difference is it does start building its MAC address table by reading the source information, then dropping the rest of the regular data frames. Lastly the root and designated ports are in the forwarding state which do send and receive ethernet frames and looks inside the payload. 

I configured VLAN 10,20 on the 3560-x to be the primary (root bridge) and VLAN 30,40 as secondary (backup root bridge). I changed the 3560-x to have a bridge priority of 0 for VLAN 10,20 and a priority of 4096 for VLAN 30,40. While we have vice-versa on the 3750-x with VLAN 30,40 as primary and VLAN 10,20 as secondary and the same priorities flipped. 

In this example we are going to discuss the logic behind STP and how our topology can be loop free, and mitigating broadcast storms. In VLAN 10's STP process the 3560-x is the root bridge with a priority of 10, which is the 0 + the extended system ID which is VLAN 10. 3750-x has a priority of 4106, 2960_1 at 32778 and 2960_2 at 32778. With this in mind we are able to find the lowest BID easily with show commands. The next step is finding one root port for each switch with the rule of lowest root cost, lowest neighbor BID, and last tie-breaker being lowest neighbor port ID. Root cost is dependent on Mbps/Gbps of a link. 10Mbps is a cost of 100, while 10Gbps is a cost of 2. We are looking for the lowest root cost link towards the root bridge and since we have etherchannel, there are quite a few tie-breakers through the process because two links are going to the same switch which would be in our case the same cost, same neighbor bridge ID, then our last tie-breaker will give us our answer. The next step is finding Designated and Blocked/Alternate ports for each collision domain or segment that doesn't have a Root port. The rule of thumb is the interface on the switch with lowest root cost and the tie-breaker being interface on switch with lowest bridge ID. If you look between 2960_1 and 3750-x (this port-channel doesn't have a root port) we notice that the 3750-x's root cost to the root bridge is a cost of 4, and the 2960_x has the same cost of 4 to the root bridge. The interface on switch with lowest bridge ID is 3750-x which takes the Designated role for the Po1 and the 2960_1 end will get the Blocked role. 

<img width="635" height="363" alt="Screenshot 2026-08-20 233338" src="https://github.com/user-attachments/assets/97978601-7359-47f1-b1fb-12a49b57794c" />
<img width="636" height="374" alt="Screenshot 2026-08-20 233435" src="https://github.com/user-attachments/assets/824acb58-8d56-4916-954a-2fde67d8609f" />
<img width="631" height="370" alt="Screenshot 2026-08-20 233556" src="https://github.com/user-attachments/assets/ab077242-296f-4d70-bee3-9f85dc8a0158" />
<img width="631" height="349" alt="Screenshot 2026-08-20 233647" src="https://github.com/user-attachments/assets/36083ae5-aac8-4f6d-85b0-58c14d99e428" />














## Distribution Layer

## Core Layer
