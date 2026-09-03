# Access control matrix

> **Implementation status: mixed — some of this is now real.**
>
> Two separate enforcement points exist in this build, and they don't talk to
> each other: **SW-CORE01** (Packet Tracer, all SVIs, no ACLs — office traffic
> routes locally, bypassing inspection) and **OPNsense** (VirtualBox, a
> genuine firewall with three interfaces matching the server/mgmt/dmz
> subnets). Office VLANs (10/20/30) only exist in Packet Tracer and have no
> path to OPNsense, so any policy involving them is still design-only. But
> traffic **between the server, mgmt and DMZ zones inside VirtualBox does
> cross OPNsense for real**, and P-005/P-006/P-007 below are now implemented
> and verified there — see
> [`../04-firewall/README.md`](../04-firewall/README.md) for the actual rules
> (named FW-001 through FW-005) and live cross-zone routing tests.
>
> **Also note:** the policy below was written assuming a dedicated DMZ host
> (`srv-web01` at 192.168.60.10). In the build, the portal (Nginx) ended up
> co-located with internal DNS on `srv-dns01` in the server zone instead, and
> **VLAN 60 has no host on it.** P-007–P-009 are implemented at the firewall
> level (OPNsense has the OPT3/DMZ interface and rules ready) but have no real
> DMZ host to originate or receive that traffic yet.

## Intended policy

| ID | Source | Destination | Service | Action | Status |
|---|---|---|---|---|---|
| P-001 | Office (10/20/30) | srv-dns01 (192.168.40.10) | TCP/UDP 53 | Permit | Design only — office traffic doesn't reach either enforcement point |
| P-002 | Office (10/20/30) | Server zone (40) | Any other | Deny + log | Design only — same reason |
| P-003 | Office (10/20/30) | Ops zone (50) | Any | Deny + log | Design only — same reason |
| P-004 | Office (10/20/30) | Device mgmt (99) | Any | Deny + log | Design only — same reason |
| P-005 | Ops (50) | Server (40) | SSH, HTTPS, monitoring | Permit | **Enforced — OPNsense FW-002, verified live** |
| P-006 | Ops (50) | Device mgmt (99) | SSH | Permit | Design only — VLAN 99 (switch mgmt) is a different network than OPNsense's zones; not the same rule as FW-002 |
| P-007 | DMZ (60) | srv-log01, srv-dns01 | Approved log / DNS ports | Permit | **Rule implemented — OPNsense FW-004** — but no DMZ host exists yet to generate this traffic |
| P-008 | DMZ (60) | Server zone (40) | Any other | Deny + log | Same as P-007 — no DMZ host to test against yet |
| P-009 | DMZ (60) | Office (10/20/30) | Any | Deny + log | **Rule implemented — OPNsense FW-005** — same caveat, plus office side is unreachable regardless |
| P-010 | Any | Device mgmt (99) | Any | Deny + log | Design only — VLAN 99 is switch-side, not part of OPNsense's zones |

## Where the real risk sits now

The original design put the internet-facing portal in its own zone (DMZ) so
that if it were compromised, the blast radius stopped at P-008/P-009. In the
build, the portal runs as Nginx **on `srv-dns01` itself** — the same host, and
the same VLAN (40), as internal DNS and (per the addressing plan) alongside
`srv-mon01` and `srv-log01`.

That changes the risk in a concrete way: a compromise of the portal is no
longer a DMZ incident that the network can contain — it is immediately a
**root-level compromise of the internal DNS server**, with no network boundary
in between, because both services run as processes on the same machine. P-002
(office cannot reach the server zone) still matters, but it no longer protects
DNS from the portal, because there is no zone boundary between them at all.

**Planned fix:** move Nginx to a dedicated host in VLAN 60 (reviving the
original `srv-web01` plan), or, if resourcing stays tight, at minimum run
Nginx in a container or under a restricted service account with no access to
the DNS zone files or `named` configuration, so a web-layer compromise cannot
walk directly into the DNS service on the same box.

## Planned enforcement

Inbound extended ACLs on the office SVI, starting with the highest-value rules
(the DMZ SVI is deferred until a host actually exists in VLAN 60):

```
ip access-list extended ACL-OFFICE-IN
 remark P-001 permit DNS to internal resolver
 permit udp 192.168.10.0 0.0.0.255 host 192.168.40.10 eq 53
 permit tcp 192.168.10.0 0.0.0.255 host 192.168.40.10 eq 53
 remark P-003 / P-004 deny office to management zones
 deny   ip 192.168.10.0 0.0.0.255 192.168.50.0 0.0.0.255 log
 deny   ip 192.168.10.0 0.0.0.255 192.168.99.0 0.0.0.255 log
 remark P-002 deny office to server zone
 deny   ip 192.168.10.0 0.0.0.255 192.168.40.0 0.0.0.255 log
 permit ip any any
!
interface Vlan10
 ip access-group ACL-OFFICE-IN in
```

Verification for each rule is a paired test: confirm the permitted path still
works, and confirm the denied path fails. Testing only the permit side is how
policies end up looking correct while blocking nothing.
