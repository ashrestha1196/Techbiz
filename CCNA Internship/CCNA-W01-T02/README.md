
# CCNA Week 1 — Task 2: IPv4 Subnetting and VLSM

## Objective
Design and implement a VLSM addressing scheme using
192.168.10.0/24 in Cisco Packet Tracer.

## Network Topology
- 2 Cisco routers
- 2 switches
- 4 PCs
- 4 VLANs (10, 11, 12, 13)
- OSPF Area 0

## VLSM Addressing

| VLAN | Network | Hosts Required |
|------|---------|----------------|
| 10 | 192.168.10.0/26 | 50 |
| 11 | 192.168.10.64/27 | 25 |
| 12 | 192.168.10.96/28 | 12 |
| 13 | 192.168.10.112/30 | 2 |
| R1-R2 | 192.168.10.116/30 | 2 |

## Implementation
1. Created VLANs and configured access ports.
2. Configured 802.1Q trunk ports.
3. Configured router subinterfaces.
4. Assigned IP addresses and default gateways.
5. Enabled OSPF Area 0 on both routers.

## Verification
- OSPF neighbor adjacency reached FULL state.
- Remote VLSM routes learned successfully.
- All four PCs communicated successfully.

## Key Learning
VLANs provide Layer 2 segmentation, while VLSM
provides efficient Layer 3 address allocation.

OSPF dynamically exchanges routing information
between routers.

## Evidence
See the screenshots folder and Packet Tracer file.
