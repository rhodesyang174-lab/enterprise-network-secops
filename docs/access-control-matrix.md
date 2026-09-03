# Access control matrix

> **Implementation status: designed, not enforced.**
>
> Every SVI in this build lives on SW-CORE01, so inter-VLAN traffic is routed
> locally and does not pass any inspection point. The table below is the intended
> policy. It is documented here because the design work is real and the gap is
> worth being explicit about — not because it is currently active.
>
> **Also note:** the policy below was written assuming a dedicated DMZ host
> (`srv-web01` at 192.168.60.10). In the build, the portal (Nginx) ended up
> co-located with internal DNS on `srv-dns01` in the server zone instead, and
> **VLAN 60 has no host on it.** Rules P-007–P-009 describe intent that no
> longer maps to the real topology — see the note below the table.
>
> Enforcing the reachable parts of this matrix is the top item on the
> remediation list (README, limitation 1).

## Intended policy

| ID | Source | Destination | Service | Action | Rationale |
|---|---|---|---|---|---|
| P-001 | Office (10/20/30) | srv-dns01 (192.168.40.10) | TCP/UDP 53 | Permit | Internal name resolution |
| P-002 | Office (10/20/30) | Server zone (40) | Any other | Deny + log | Workstations have no business reaching servers directly |
| P-003 | Office (10/20/30) | Ops zone (50) | Any | Deny + log | Keeps user devices out of the management zone |
| P-004 | Office (10/20/30) | Device mgmt (99) | Any | Deny + log | Management plane is not reachable from user VLANs |
| P-005 | Ops (50) | Server (40) | SSH, HTTPS, monitoring | Permit | Authorized administration |
| P-006 | Ops (50) | Device mgmt (99) | SSH | Permit | Switch management from the admin workstation only |
| P-007 | ~~DMZ (60)~~ | ~~srv-log01, srv-dns01~~ | ~~Approved log / DNS ports~~ | — | **Not applicable — no host in VLAN 60. See note.** |
| P-008 | ~~DMZ (60)~~ | ~~Server zone (40)~~ | ~~Any other~~ | — | **Not applicable — see note.** |
| P-009 | ~~DMZ (60)~~ | ~~Office (10/20/30)~~ | ~~Any~~ | — | **Not applicable — see note.** |
| P-010 | Any | Device mgmt (99) | Any | Deny + log | Default-deny to the management VLAN |

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
