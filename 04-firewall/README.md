# 04 · Edge firewall — zones, NAT, access control

> **TODO** — fill in the blanks marked below and drop screenshots into `evidence/`.

**Task objective.** Enforce the access-control matrix at the network edge:
interface zoning, source NAT for internet egress, destination NAT for the portal,
and management access restriction.

Platform: `TODO: OPNsense / pfSense / other — and version`

## Interfaces

| Interface | Zone | Address | Connects to |
|---|---|---|---|
| WAN | Untrusted | `TODO` | Simulated internet |
| LAN | Office | `TODO` | |
| SERVER | Server zone | `TODO` | |
| MGMT | Operations | `TODO` | |
| DMZ | DMZ | `TODO` | |

## NAT

**Source NAT** — office zones to the internet, permitting DNS, HTTP, HTTPS and
explicitly approved ports only.

**Destination NAT** — simulated public address to srv-web01 (192.168.60.10) on
port 80/443, with a matching filter rule. A port-forward without a corresponding
rule is the classic "the NAT is right but nothing works" case.

## Policy

Implemented from [`../docs/access-control-matrix.md`](../docs/access-control-matrix.md).

| ID | Source | Destination | Service | Action | Implemented | Verified |
|---|---|---|---|---|---|---|
| P-001 | Office | srv-dns01 | TCP/UDP 53 | Permit | | |
| P-002 | Office | Server zone | Any other | Deny + log | | |
| P-003 | Office | Ops zone | Any | Deny + log | | |
| P-005 | Ops | Server, DMZ | SSH, HTTPS | Permit | | |
| P-007 | DMZ | srv-log01, srv-dns01 | Approved ports | Permit | | |
| P-008 | DMZ | Server zone | Any other | Deny + log | | |
| P-009 | DMZ | Office | Any | Deny + log | | |

> **Important:** with all SVIs on the core switch (section 01), traffic between
> internal zones never reaches this firewall. Only flows that actually transit
> the edge can be enforced here. `TODO: state which of the rules above are
> genuinely enforced in your topology and which are not — this is the honest
> version and it is worth more than a table of ten green checkmarks.`

## Change discipline

Every policy change recorded with: reason, blast radius, pre-change config
export, implementation window, and rollback trigger. Post-change testing covers
**both** directions — the permitted path works *and* the denied path fails.
Testing only the permit side is how a policy ends up looking correct while
blocking nothing.

## Verification

| Test | Expected | Result | Evidence |
|---|---|---|---|
| Office workstation reaches the internet | Approved ports only | | |
| Simulated external client reaches the portal | HTTP 200 via DNAT | | |
| Office workstation reaches the ops zone | Denied, logged | | |
| DMZ host initiates a connection to an office host | Denied, logged | | |

## Known limitations

`TODO — e.g. which matrix rules are unenforceable in the current topology? is
firewall management restricted by source? are configs exported and version
controlled?`
