# VLAN Segmentation, Broadcast Domains & Ethernet Frame Forwarding

**Tech Biz Academy — CCNA Networking Internship | Week 2, Task 1 (`CCNA-W02-T01`)**

A Cisco Packet Tracer lab exploring how VLANs divide a physical switch into separate Layer 2 broadcast domains, how Ethernet frames are forwarded, and why a failed ping can have different causes.

## Project Overview

I connected six PCs to a Cisco Layer 2 switch, created three VLANs, assigned access ports, and verified that PCs could communicate within their VLANs but not across VLANs without routing.

I then used Packet Tracer Simulation Mode to investigate ARP, MAC address learning, VLAN-scoped flooding, and the difference between a host's ARP cache and a switch's MAC address table.

### Objectives

- Explain broadcast domains and the limitations of a flat Layer 2 network.
- Create VLAN **10 (Staff)**, **20 (Finance)**, and **30 (Guest)**.
- Configure access ports and assign two PCs to each VLAN.
- Verify same-VLAN connectivity and failed cross-VLAN connectivity.
- Interpret `show vlan brief` and `show mac address-table`.
- Trace Ethernet and ARP forwarding at Layer 2.
- Investigate VLAN isolation and unknown-unicast flooding.

## Tools and Topology

- **Simulator:** Cisco Packet Tracer
- **Switch:** One Cisco Layer 2 switch (Switch0)
- **Endpoints:** Six PCs (PC0–PC5)
- **Routing:** None configured

### Network Topology

```text
                        +--------------------+
                        |   Switch0 (L2)     |
                        |     Cisco 2960     |
                        +--------------------+
                          |   |   |   |   |   |
                          |   |   |   |   |   |
       VLAN 10 (Staff) ---+---+   |   |   |   |
           Fa0/1: PC0             |   |   |   |
           Fa0/2: PC1             |   |   |   |
                                 |   |   |   |
       VLAN 20 (Finance) --------+---+   |   |
           Fa0/3: PC2                     |   |
           Fa0/4: PC3                     |   |
                                         |   |
       VLAN 30 (Guest) ------------------+---+
           Fa0/5: PC4
           Fa0/6: PC5
```

**Topology screenshot:**
![Packet Tracer VLAN Topology](03-Screenshots/01-topology.png)

### VLAN and IP Addressing Plan

| VLAN | Department | Device | Switch Port | IPv4 Address | Subnet Mask |
|------|------------|--------|-------------|--------------|-------------|
| 10 | Staff | PC0 | Fa0/1 | 192.168.10.11 | 255.255.255.0 |
| 10 | Staff | PC1 | Fa0/2 | 192.168.10.12 | 255.255.255.0 |
| 20 | Finance | PC2 | Fa0/3 | 192.168.20.11 | 255.255.255.0 |
| 20 | Finance | PC3 | Fa0/4 | 192.168.20.12 | 255.255.255.0 |
| 30 | Guest | PC4 | Fa0/5 | 192.168.30.11 | 255.255.255.0 |
| 30 | Guest | PC5 | Fa0/6 | 192.168.30.12 | 255.255.255.0 |

Default gateways were left blank during the initial isolation tests because no inter-VLAN routing device was present.

---

## 1. Baseline: Default Broadcast Domain

Before configuring VLANs, I executed:

```cisco
show vlan brief
```

All switch access interfaces initially belonged to **VLAN 1 (`default`)**.

Creating additional VLANs did not automatically move any ports from VLAN 1.

**Evidence:** `screenshots/02-default-vlan.png`

<!-- ![Default VLAN Configuration](screenshots/02-default-vlan.png) -->

## 2. VLAN Creation and Access Port Configuration

I created three VLANs and assigned two PC-facing interfaces to each.

### VLAN Configuration

```cisco
enable
configure terminal

vlan 10
 name staff

vlan 20
 name finance

vlan 30
 name guest
```

### Access Port Configuration

```cisco
interface range fa0/1 - 2
 switchport mode access
 switchport access vlan 10

interface range fa0/3 - 4
 switchport mode access
 switchport access vlan 20

interface range fa0/5 - 6
 switchport mode access
 switchport access vlan 30

end
show vlan brief

copy running-config startup-config
```

### Verification

| VLAN | Name | Assigned Interfaces |
|------|------|---------------------|
| 10 | staff | Fa0/1, Fa0/2 |
| 20 | finance | Fa0/3, Fa0/4 |
| 30 | guest | Fa0/5, Fa0/6 |

The configuration save returned `[OK]`.

### Commands Explained

- `switchport mode access` — Configures the interface as an access port.
- `switchport access vlan <id>` — Assigns the access port to a VLAN.
- `show vlan brief` — Displays VLAN IDs, names, operational status, and access-port membership.
- `copy running-config startup-config` — Saves the active configuration.

An access port normally transmits untagged Ethernet frames. The switch determines their VLAN membership from the port configuration.

**Evidence:**
- `screenshots/03-vlans-created.png`
- `screenshots/04-access-ports.png`
- `screenshots/09-final-verification.png`

---

## 3. Connectivity Tests

### Same-VLAN Connectivity — Successful

| Source | Destination | Result |
|--------|-------------|--------|
| PC0 / VLAN 10 | PC1 / `192.168.10.12` | 4/4 replies, 0% loss |
| PC2 / VLAN 20 | PC3 / `192.168.20.12` | 4/4 replies, 0% loss |
| PC4 / VLAN 30 | PC5 / `192.168.30.12` | 4/4 replies, 0% loss |

These results confirmed that devices within each VLAN could communicate.

**Evidence:** `screenshots/05-same-vlan-pings.png`

<!-- ![Same VLAN Ping Results](screenshots/05-same-vlan-pings.png) -->

### Cross-VLAN Connectivity — Failed

| Source | Destination | Result |
|--------|-------------|--------|
| PC0 / VLAN 10 | PC2 / `192.168.20.11` | 4/4 timed out, 100% loss |
| PC0 / VLAN 10 | PC4 / `192.168.30.11` | 4/4 timed out, 100% loss |

**Important observation:** These failed pings also reflected the absence of a default gateway. They did not independently prove VLAN isolation at Layer 2.

I investigated this further using a controlled ARP experiment.

**Evidence:** `screenshots/06-cross-vlan-pings.png`

<!-- ![Cross VLAN Ping Failures](screenshots/06-cross-vlan-pings.png) -->

---

## 4. Additional Experiment: Proving Layer 2 Isolation Using ARP

To isolate the VLAN effect from IP routing, I temporarily changed **PC2**, which remained in **VLAN 20**, to:

`192.168.10.21/24`

PC0 remained in VLAN 10 with:

`192.168.10.11/24`

Both PCs were now configured in the same IP subnet but belonged to different VLANs.

### Packet Tracer Simulation Mode Observations

1. PC0 identified `192.168.10.21` as a local destination.
2. PC0 generated an ARP request to discover the destination MAC address.
3. The Ethernet frame used the broadcast MAC address `FFFF.FFFF.FFFF`.
4. Switch0 received the frame on **Fa0/1 in VLAN 10**.
5. The switch flooded the broadcast to **Fa0/2**, not Fa0/3 in VLAN 20.
6. PC1 received the ARP request but did not own the requested IP address, so it did not reply.
7. PC0 received no ARP reply.
8. ARP resolution timed out, and the pending ICMP packet was dropped.

### Finding

Even when devices are configured with IP addresses from the same subnet, a Layer 2 switch does not forward ordinary broadcast traffic across VLAN boundaries.

This directly demonstrated VLAN isolation at Layer 2.

### Additional Troubleshooting Observations

I compared three different connectivity failures:

| Scenario | Observation | Reason |
|----------|-------------|--------|
| Remote subnet, no default gateway | Ping failed | No valid next hop |
| Same subnet, different VLANs | ARP request timed out | VLAN boundary prevented ARP delivery |
| Remote subnet, unreachable gateway | ARP for gateway failed | Gateway MAC could not be resolved |

**Key lesson:** A failed `ping` is a symptom, not a root-cause diagnosis.

**Evidence:** Add selected Packet Tracer PDU screenshots to `screenshots/07-arp-isolation/`.

---

## 5. MAC Address Table Verification

After generating traffic within each VLAN, I executed:

```cisco
show mac address-table dynamic
```

### Observed MAC Address Table

| VLAN | MAC Address | Type | Port |
|------|-------------|------|------|
| 10 | `000c.cf52.1a22` | DYNAMIC | Fa0/1 |
| 10 | `0030.a30d.5a22` | DYNAMIC | Fa0/2 |
| 20 | `0001.c961.ac8c` | DYNAMIC | Fa0/4 |
| 20 | `00d0.ba87.b4a1` | DYNAMIC | Fa0/3 |
| 30 | `0000.0ccc.c6ab` | DYNAMIC | Fa0/6 |
| 30 | `0060.2f7e.2233` | DYNAMIC | Fa0/5 |

### MAC Table Columns Explained

| Column | Meaning |
|--------|---------|
| Vlan | VLAN associated with the MAC address entry |
| Mac Address | MAC address learned by the switch |
| Type | Dynamic or static learning method |
| Ports | Interface associated with the learned MAC address |

Dynamic MAC entries are learned from the source MAC addresses of incoming Ethernet frames.

**Evidence:** `screenshots/08-mac-table.png`

<!-- ![MAC Address Table](screenshots/08-mac-table.png) -->

---

## 6. Additional Experiment: Cached ARP and Unknown-Unicast Flooding

I investigated what happens when the sending PC knows a destination MAC address, but that destination belongs to another VLAN.

### Experiment

I temporarily moved PC2's Fa0/3 interface into VLAN 10 and communicated with PC2 from PC0.

PC0 learned this ARP mapping:

```text
192.168.10.21 -> 00d0.ba87.b4a1
```

I then moved PC2's interface back into VLAN 20.

PC0 still retained its ARP cache entry, allowing it to send an Ethernet unicast frame without performing another ARP request.

However, Switch0 learned PC2's MAC address in VLAN 20:

```text
20    00d0.ba87.b4a1    DYNAMIC    Fa0/3
```

When PC0 sent the Ethernet frame from VLAN 10, Switch0 could not find a matching destination MAC address **within VLAN 10**.

The switch therefore treated the frame as an unknown unicast and flooded it only through eligible ports belonging to VLAN 10.

### What I Learned

- A PC's ARP cache maps **IP address → MAC address**.
- A switch's MAC address table maps **(VLAN, MAC address) → interface**.
- MAC address learning is VLAN-aware.
- A destination MAC address learned in a different VLAN cannot be used for ordinary Layer 2 forwarding from the current VLAN.
- Unknown-unicast flooding occurs within the frame's VLAN.

This experiment helped me understand that knowing a destination MAC address does not bypass VLAN separation.

---

## 7. How a Switch Processes an Ethernet Frame

Based on the practical investigation, I learned that Ethernet switching follows these general steps:

1. **Frame Reception:** The switch receives an Ethernet frame on an ingress interface.
2. **VLAN Classification:** The switch identifies the VLAN using the port configuration or an 802.1Q tag.
3. **Source MAC Learning:** The switch learns or refreshes the source MAC address and associates it with the VLAN and ingress interface.
4. **Destination Inspection:** The switch examines the destination MAC address.
5. **MAC Table Lookup:** For a unicast frame, the switch searches for the destination MAC within the relevant VLAN.
6. **Forwarding:** The switch determines the eligible outgoing interface or interfaces.

### Forwarding Behavior

| Frame Type | Switch Behavior |
|------------|-----------------|
| Broadcast | Floods within the VLAN |
| Known unicast | Forwards to the learned destination port |
| Unknown unicast | Floods within the VLAN |
| Multicast | May flood or use multicast forwarding information |

A standard Layer 2 switch does not perform inter-VLAN IP routing.

Communication between VLANs normally requires a router or Layer 3 switch, with a separate IP subnet assigned to each VLAN.

---

## 8. Why Flat Broadcast Domains Do Not Scale Well

A flat Layer 2 network can consist of multiple interconnected switches while still sharing one broadcast domain.

As the network grows, several challenges may increase:

- Unnecessary broadcast traffic processing
- Larger impact from broadcast storms and Layer 2 problems
- More complex troubleshooting
- Limited departmental separation
- Difficulty applying different access policies

VLANs help by dividing the network into logical broadcast domains.

However, VLAN segmentation does not replace:

- Spanning Tree Protocol (STP)
- Access Control Lists (ACLs)
- Firewalls
- Adequate physical bandwidth
- Proper switch configuration and monitoring

For example, a broadcast storm in VLAN 10 will not normally be forwarded into VLANs 20 and 30, but it can still indirectly affect them through shared physical resources.

---

## 9. Key Learning Reflection

Before this exercise, I understood Ethernet switching primarily as a MAC-address-based operation.

Because all interfaces on a default switch belong to VLAN 1, I had not fully recognized the importance of VLAN membership in forwarding decisions.

After configuring VLANs and examining frames in Packet Tracer Simulation Mode, I understood that **the switch uses VLAN context together with MAC address information when making forwarding decisions**.

The ARP experiment demonstrated how a broadcast remains confined to its VLAN.

The additional MAC-table experiment demonstrated why a MAC address learned in another VLAN cannot be used for ordinary Layer 2 forwarding.

I also learned that two failed pings can have completely different underlying causes.

Understanding packet flow and protocol behavior is more valuable for troubleshooting than simply determining whether a ping succeeded or failed.

---


