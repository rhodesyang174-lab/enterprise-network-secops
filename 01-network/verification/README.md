# Verification log

Connectivity and policy tests run against the deployed topology.

> Fill in the Result column and drop the matching screenshots into this folder.
> Name them by test ID, e.g. `NET-01-dhcp-lease.png`.

## Connectivity

| ID | Test | Expected | Result | Evidence |
|---|---|---|---|---|
| NET-01 | Workstation obtains a DHCP lease | Address in the correct subnet, gateway `.1`, DNS 192.168.40.10 |<img width="863" height="699" alt="image" src="https://github.com/user-attachments/assets/ad44f4eb-52ed-4445-ba78-6954dab6caea" />
 | |
| NET-02 | Workstation reaches its own gateway | Ping succeeds |<img width="775" height="400" alt="image" src="https://github.com/user-attachments/assets/23f8d45f-d4ef-414b-87a8-58e7357fc19f" />
 | |
| NET-03 | Office workstation reaches a host in another office VLAN | Ping succeeds via core routing |<img width="875" height="889" alt="image" src="https://github.com/user-attachments/assets/ec657240-edc1-4d45-8eb2-0e3fd52ae29e" />
 | |
| NET-04 | Office workstation reaches the server zone | Ping succeeds (no policy enforced yet — see note below) |<img width="875" height="889" alt="image" src="https://github.com/user-attachments/assets/e79c5563-8569-42ff-bb38-9af3b117b057" />
 | |
| NET-05 | Trunk carries all seven VLANs | `show interfaces trunk` lists 10,20,30,40,50,60,99 on both ends |<img width="876" height="664" alt="image" src="https://github.com/user-attachments/assets/846ac992-a213-4abd-8ae1-bbcb2bb95b3a" />
 | |
| NET-06 | Access switch reachable over SSH from the ops workstation | SSH to 192.168.99.11 / .12 succeeds |<img width="876" height="889" alt="image" src="https://github.com/user-attachments/assets/78f45dcd-ca61-4358-87dc-1649935c3f16" />
 | |
| NET-07 | Telnet to a switch is refused | Connection rejected on all VTY lines |<img width="876" height="889" alt="image" src="https://github.com/user-attachments/assets/c649a006-0dd5-4c47-8041-530251b0fb00" />
 | |

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
