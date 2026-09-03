# Verification log

Connectivity and policy tests run against the deployed topology.

> Fill in the Result column and drop the matching screenshots into this folder.
> Name them by test ID, e.g. `NET-01-dhcp-lease.png`.

## Connectivity

| ID | Test | Expected | Result | Evidence |
|---|---|---|---|---|
| NET-01 | Workstation obtains a DHCP lease | Address in the correct subnet, gateway `.1`, DNS 192.168.40.10 | | |
| NET-02 | Workstation reaches its own gateway | Ping succeeds | | |
| NET-03 | Office workstation reaches a host in another office VLAN | Ping succeeds via core routing | | |
| NET-04 | Office workstation reaches the server zone | Ping succeeds (no policy enforced yet — see note below) | | |
| NET-05 | Trunk carries all seven VLANs | `show interfaces trunk` lists 10,20,30,40,50,60,99 on both ends | | |
| NET-06 | Access switch reachable over SSH from the ops workstation | SSH to 192.168.99.11 / .12 succeeds | | |
| NET-07 | Telnet to a switch is refused | Connection rejected on all VTY lines | | |

## Notes on NET-04

This test currently **succeeds**, and that is the finding. Under the intended
access-control matrix (P-002) it should fail. It succeeds because all SVIs are
on the core and no ACL is applied, so office-to-server traffic is routed locally
without inspection.

Recording it as a passing ping rather than a policy failure would misrepresent
the build. Once the planned ACLs are applied, this row becomes a deny test and
the expected result inverts.

## Useful verification commands

```
show vlan brief
show interfaces trunk
show ip interface brief
show ip route
show ip dhcp binding
show running-config | section vty
```
