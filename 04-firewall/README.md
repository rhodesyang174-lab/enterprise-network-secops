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

**Destination NAT — open design question, not yet resolved.** The plan
originally called for the public address to forward to `srv-web01`
(192.168.60.10) in the DMZ. In the build, the portal runs on `srv-dns01`
(192.168.40.10) in the **server zone** instead, and VLAN 60 has no host on it
— see [`../docs/access-control-matrix.md`](../docs/access-control-matrix.md).

That leaves two real options, and `TODO: which one did you actually implement
(or decide against)?`:

1. **Forward the public address into the server zone** (192.168.40.10). This
   makes the portal reachable from outside, but it means opening an inbound
   WAN path directly into the same zone as internal DNS, monitoring and log
   collection — a materially worse security posture than the original DMZ
   design, since a compromised portal now has no zone boundary between it and
   those services either from the network side or, as noted in
   `access-control-matrix.md`, the host side.
2. **Don't forward at all** — treat the portal as internal-only for this build
   (reachable from office/ops zones, not from the internet), and note that
   public exposure is deferred until a dedicated DMZ host exists.

Whichever was chosen, state it here plainly rather than leaving the NAT rule
implicit — this is exactly the kind of decision a reviewer will ask about.

## Policy

Implemented from [`../docs/access-control-matrix.md`](../docs/access-control-matrix.md).

| ID | Source | Destination | Service | Action | Implemented | Verified |
|---|---|---|---|---|---|---|
| P-001 | Office | srv-dns01 | TCP/UDP 53 | Permit | | |
| P-002 | Office | Server zone | Any other | Deny + log | | |
| P-003 | Office | Ops zone | Any | Deny + log | | |
| P-005 | Ops | Server zone | SSH, HTTPS | Permit | | |
| P-007–009 | ~~DMZ~~ | — | — | **N/A — no host in VLAN 60, see `../docs/access-control-matrix.md`** | | |

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
| Simulated external client reaches the portal | HTTP 200 via DNAT — **only if option 1 above was chosen; if option 2, mark N/A and note it as intentional** | | |
| Office workstation reaches the ops zone | Denied, logged | | |
| DMZ host initiates a connection to an office host | Denied, logged | | |

## Known limitations

`TODO — e.g. which matrix rules are unenforceable in the current topology? is
firewall management restricted by source? are configs exported and version
controlled?`
