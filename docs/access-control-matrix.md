# Access control matrix

> **Implementation status: designed, not enforced.**
>
> Every SVI in this build lives on SW-CORE01, so inter-VLAN traffic is routed
> locally and does not pass any inspection point. The table below is the intended
> policy. It is documented here because the design work is real and the gap is
> worth being explicit about — not because it is currently active.
>
> Enforcing it is the top item on the remediation list (README, limitation 1).

## Intended policy

| ID | Source | Destination | Service | Action | Rationale |
|---|---|---|---|---|---|
| P-001 | Office (10/20/30) | srv-dns01 (192.168.40.10) | TCP/UDP 53 | Permit | Internal name resolution |
| P-002 | Office (10/20/30) | Server zone (40) | Any other | Deny + log | Workstations have no business reaching servers directly |
| P-003 | Office (10/20/30) | Ops zone (50) | Any | Deny + log | Keeps user devices out of the management zone |
| P-004 | Office (10/20/30) | Device mgmt (99) | Any | Deny + log | Management plane is not reachable from user VLANs |
| P-005 | Ops (50) | Server (40), DMZ (60) | SSH, HTTPS, monitoring | Permit | Authorized administration |
| P-006 | Ops (50) | Device mgmt (99) | SSH | Permit | Switch management from the admin workstation only |
| P-007 | DMZ (60) | srv-log01, srv-dns01 | Approved log / DNS ports | Permit | The portal needs logging and name resolution |
| P-008 | DMZ (60) | Server zone (40) | Any other | Deny + log | Limits lateral movement if the portal is compromised |
| P-009 | DMZ (60) | Office (10/20/30) | Any | Deny + log | No reverse path from an internet-facing host into user space |
| P-010 | Any | Device mgmt (99) | Any | Deny + log | Default-deny to the management VLAN |

## Why the DMZ rules matter most

VLAN 60 hosts the only service intended to be reachable from outside. If that
host is compromised, the attacker's next move is lateral — toward the server
zone or the management VLAN. P-008, P-009 and P-010 are what turn a single
compromised web server into a contained incident rather than a network-wide one.

In the current build those paths are open, because DMZ and server-zone traffic
is routed on the core with no ACL between them, and both zones share
SW-ACCESS02.

## Planned enforcement

Inbound extended ACLs on the office and DMZ SVIs, starting with the highest-value
rules:

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
