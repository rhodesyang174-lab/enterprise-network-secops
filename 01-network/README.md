# 01 · Network — VLANs, routing, DHCP

**Task objective.** Deploy the segmented Layer 2/3 foundation: VLANs, 802.1Q
trunking, inter-VLAN routing, DHCP, and switch management hardening.

## Devices

| Device | Platform | Role | Management IP |
|---|---|---|---|
| SW-CORE01 | Catalyst 3560-24PS | L3 core: inter-VLAN routing, DHCP, default route | 192.168.99.1 (VLAN 99) |
| SW-ACCESS01 | Catalyst 2960-24TT | L2 access: Admin / Marketing / R&D | 192.168.99.11 |
| SW-ACCESS02 | Catalyst 2960-24TT | L2 access: Server / Ops / DMZ | 192.168.99.12 |

Uplinks: `SW-ACCESS01 Fa0/24 ↔ SW-CORE01 Fa0/24` · `SW-ACCESS02 Fa0/24 ↔ SW-CORE01 Fa0/23`

Configurations: [`configs/`](configs/) · Test log: [`verification/`](verification/)

## Design decisions

**Routing on the core, not router-on-a-stick.** A 3560 routes between SVIs in
hardware, avoiding the single-link bottleneck where all inter-VLAN traffic
crosses one physical interface twice. The trade-off is that inter-zone traffic
never leaves the switch and therefore cannot be inspected — see limitations.

**Explicit trunk allowed-VLAN lists.** Trunks carry `10,20,30,40,50,60,99`
rather than defaulting to all VLANs. A missing VLAN in this list is one of the
most common root causes of "one department lost access to everything", so making
it explicit also makes it diagnosable.

**VLAN 1 shut down; management on VLAN 99.** No user or server port sits in the
default VLAN, and switch management addresses live in a subnet carrying no other
traffic. Compromising a workstation does not place an attacker in the same
broadcast domain as the switch management interfaces.

**SSH on all sixteen VTY lines.** Both `line vty 0 4` and `line vty 5 15` are
configured. Configuring only the first five is a common oversight that leaves
lines 5–15 accepting telnet.

## Verification

See [`verification/README.md`](verification/README.md).

## Known limitations

1. **Zone policy is designed but not enforced.** All SVIs are on the core, so
   inter-VLAN traffic is routed locally with no ACL applied. The matrix in
   `docs/access-control-matrix.md` describes intent, not enforced behaviour.
   *Next: inbound extended ACLs on the office and DMZ SVIs.*
2. **No redundancy.** Single core, single uplink per access switch — both are
   single points of failure. Defensible at 80 users, but a hardware failure is
   an outage. *Next: second core with HSRP, dual-homed access over EtherChannel.*
3. **VTY lines accept SSH from any subnet.** *Next: `access-class` permitting
   only 192.168.50.0/24.*
4. **Default native VLAN 1 on trunks.** *Next: set an unused native VLAN.*
5. **Unused access ports remain in VLAN 1 and administratively up.**
6. **Spanning tree at defaults** — no root bridge priority, no PortFast or BPDU
   Guard. No operational impact in this loop-free topology, but root bridge
   placement is not deterministic.
7. **No NTP, syslog forwarding or SNMPv3**, so device events carry no
   synchronized timestamps and are not collected anywhere. This is what section
   07 depends on.
8. **DHCP pools exist for VLANs 40/50/60/99** which are documented as statically
   addressed. Redundant, and they would hand out addresses in those zones.
