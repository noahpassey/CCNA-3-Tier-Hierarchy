<img width="676" height="339" alt="Screenshot 2026-08-20 145555" src="https://github.com/user-attachments/assets/0d6d83d8-58f7-47b7-9998-efde2bd9c250" />

# Access Layer 

I started from the beginning with creating subnetworks (segmenting at layer-3) starting with the private Class B 172.16.0.0 /12. My first network for the VLAN 10 "PC's" needed an address range of 200 hosts so I chose the /24 CIDR notation which has 253 hosts in total. My second network is VLAN 20 "WAPs" and it needed 100 hosts. The next available network address is 172.16.1.0 and I used the /25 CIDR. VLAN 30 needed 50 hosts for "Server's" on the network 172.16.1.128. VLAN 40 needed 15 hosts for "Management" purposes and is also the native untagged VLAN on trunk ports. I used the 172.16.1.192 /27 with 30 hosts in total.

This is a Dual-Stack network which consists of IPV4 and IPV6 processes at once. I started the IPV6 networks with the prefix 2000:db8:1234:VLAN Number:: /64. I used the 4th hextet to represent the VLAN number of each subnet. I converted the binary of 10,20,30,40 into hexadecimal A,14,1E,28.

2960_1 was segmented into two separate broadcast domains, VLAN 10 and 20. The purpose for this is when an ethernet header destination mac is all F's or an ipv4 header has a broadcast ip address whether for finding a neighbor or next hops mac address using ARP or finding the DHCP server with DORA messages, 




## Distribution Layer

## Core Layer
