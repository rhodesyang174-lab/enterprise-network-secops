# IP addressing and VLAN plan

## Design rules

1. One VLAN per functional zone, not per department headcount.
2. `/24` per zone — far more addresses than the 80-user requirement needs, chosen
   for readability and to leave room for growth.
3. Gateway is always `.1`.
4. `.1` through `.20` reserved for infrastructure in every subnet and excluded
   from DHCP.
5. Servers and network devices use static addresses; user workstations use DHCP.
6. Device management addresses live in a dedicated VLAN carrying no other traffic.

## Zones

| Zone | VLAN | Subnet | Gateway | DHCP range | Assignment |
|---|---|---|---|---|---|
| Administration | 10 | 192.168.10.0/24 | 192.168.10.1 | .21–.254 | DHCP |
| Marketing | 20 | 192.168.20.0/24 | 192.168.20.1 | .21–.254 | DHCP |
| R&D | 30 | 192.168.30.0/24 | 192.168.30.1 | .21–.254 | DHCP |
| Server zone | 40 | 192.168.40.0/24 | 192.168.40.1 | — | Static |
| Operations / management | 50 | 192.168.50.0/24 | 192.168.50.1 | — | Static |
| DMZ | 60 | 192.168.60.0/24 | 192.168.60.1 | — | Static |
| Device management | 99 | 192.168.99.0/24 | 192.168.99.1 | — | Static |

> DHCP pools currently exist for VLANs 40, 50, 60 and 99 as well. These are
> redundant given the static-addressing rule above and are flagged for removal —
> see limitation 10 in the top-level README.

## Planned host addressing

| Host | Address | Zone | Purpose |
|---|---|---|---|
| SW-CORE01 | 192.168.99.1 | VLAN 99 | L3 core |
| SW-ACCESS01 | 192.168.99.11 | VLAN 99 | Access switch, office |
| SW-ACCESS02 | 192.168.99.12 | VLAN 99 | Access switch, server/ops/DMZ |
| srv-dns01 | 192.168.40.10 | Server | Internal DNS |
| srv-mon01 | 192.168.40.20 | Server | Monitoring |
| srv-log01 | 192.168.40.30 | Server | Log collection |
| srv-auto01 | 192.168.50.10 | Ops | Automation control node |
| srv-web01 | 192.168.60.10 | DMZ | Company portal |
| ops-client01 | 192.168.50.100 | Ops | Administrator workstation |

All workstations receive `192.168.40.10` as their DNS server via DHCP.

## Edge

The core carries a default route to `192.168.254.1`, representing the edge
firewall. That device is not instantiated in this Packet Tracer topology, so
NAT and internet egress are out of scope for this repository.
