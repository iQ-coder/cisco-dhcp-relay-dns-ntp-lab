
# Cisco DHCP Relay / DNS / NTP Lab

## Overview
This lab demonstrates core network services on Cisco IOS using GNS3 with c3725 routers. 
The focus is on DHCP relay behavior across Layer 3 boundaries, internal DNS, and NTP synchronization.

## Topology
![Screenshot](https://raw.githubusercontent.com/iQ-coder/cisco-dhcp-relay-dns-ntp-lab/master/Screenshot%202026-06-09%20032036.png)

## Device Addressing
| Device | Interface | IP |
|--------|-----------|-----|
| R1 | fa0/0 | 10.0.0.1/30 |
| R2 | fa0/0 | 10.0.0.2/30 |
| R2 | fa0/1 | 192.168.1.1/24 |
| VPCS1/2 | eth0 | DHCP (192.168.1.0/24) |

## Concepts Demonstrated

### DHCP Relay
DHCP broadcasts don't cross Layer 3 boundaries. `ip helper-address` on R2's fa0/0 
intercepts client broadcasts and forwards them as unicast to R1. R2 stamps its own 
IP (192.168.1.1) into the `giaddr` field of the packet — this tells R1 which pool 
to assign addresses from.

### DNS
R1 acts as an internal DNS server using `ip dns server` and static `ip host` records. 
Clients receive R1's IP as their DNS server via DHCP option 6.

### NTP
R1 is configured as NTP master (stratum 1). R2 syncs to R1 and operates at stratum 2. 
Verified via `show ntp status` showing clock synchronized and reference 10.0.0.1.

## Key Commands
```
# DHCP Server (R1)
ip dhcp excluded-address 192.168.1.1
ip dhcp pool CLIENTS
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 10.0.0.1

# DNS (R1)
ip dns server
ip host R1 10.0.0.1
ip host R2 192.168.1.1

# NTP (R1)
ntp master 1

# NTP (R2)
ntp server 10.0.0.1

# Relay (R2)
interface fa0/0
 ip helper-address 10.0.0.1
```

## Wireshark Captures
| File | What it shows |
|------|--------------|
| `captures/dhcp-client-side.pcap` | DHCP DORA exchange — client broadcasting Discover (src 0.0.0.0 dst 255.255.255.255) |
![Screenshot](https://raw.githubusercontent.com/iQ-coder/cisco-dhcp-relay-dns-ntp-lab/master/Screenshot%202026-06-09%20023309.png)


| `captures/dhcp-relay-side.pcap` | Relay forwarding as unicast — giaddr stamped as 192.168.1.1, full DORA between R2 and 
https://github.com/iQ-coder/cisco-dhcp-relay-dns-ntp-lab/blob/master/Screenshot%202026-06-09%200233091.png
R1 |
| `captures/ntp-sync.pcap` | NTP request/response — R2 polling R1 periodically for time synchronization |

## Tools
- GNS3 2.2.59 with GNS3 VM
- Cisco c3725 IOS (adventerprisek9-mz.124-15.T14)
- VPCS client simulator
- Wireshark

## Lab Series
This lab is part of an ongoing enterprise networking series:

| Repo | Concepts |
|------|----------|
| `network-security-dmz-architecture-lab` | DMZ, VLANs, ACLs, static routing, SSH |
| `cisco-ospf-nat-dmz-enterprise-lab` | Multi-area OSPF, NAT/PAT, SSH |
| `cisco-hsrp-stp-redundancy-lab` | HSRP, Rapid PVST+, per-VLAN STP |
| `cisco-dhcp-relay-dns-ntp-lab` | DHCP relay, DNS, NTP |
