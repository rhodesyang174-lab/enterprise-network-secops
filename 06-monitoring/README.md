# 06 · Monitoring — Zabbix

> **TODO** — fill in the blanks marked below and drop screenshots into `evidence/`.

**Task objective.** Get servers, network devices and the portal service under
monitoring, with alerts that lead to a documented response rather than a
notification nobody acts on.

Platform: Zabbix `TODO: version` on srv-mon01 (192.168.40.20)

## Onboarding

**Linux hosts** via Zabbix Agent 2. The host name in the agent config must match
the host name in the console — a mismatch produces a host that exists but never
receives data.

```bash
systemctl enable --now zabbix-agent2
ss -lntp | grep 10050
journalctl -u zabbix-agent2 -n 50
```

**Network devices** via SNMP, with the source address restricted to the
monitoring server. `TODO: SNMPv3, or v2c with an ACL? Say which and why — if the
Packet Tracer platform only supports v2c, that is a real constraint worth
stating rather than glossing over.`

## Monitored hosts

| Host | Method | Template | Key metrics |
|---|---|---|---|
| srv-web01 | Agent 2 | Linux by Zabbix agent | CPU, memory, disk, Nginx process/port |
| srv-dns01 | Agent 2 | Linux by Zabbix agent | CPU, memory, disk, named process |
| SW-CORE01 | SNMP | Generic SNMP | Reachability, interface status, throughput, errors |
| `TODO` | | | |

## Alert thresholds

| Object | Condition | Response |
|---|---|---|
| CPU | Sustained above 85% | Identify the process, judge business impact |
| Memory | Sustained above 90% | Check processes, cache, load |
| Disk | Above 85% | Check logs, temp files, old backups |
| Nginx | Process down or port unreachable | Check process, config, port, error log |
| Network device | Consecutive probe failures | Check device, interface, link |
| SSH auth | Abnormal failure rate | Check account, source, auth log |

Thresholds are a starting point, not a truth. `TODO: did any of these fire
spuriously and get tuned? Recording a threshold you had to change is more
convincing than a table you never tested.`

## Alert drill

A monitoring system that has never fired is untested.

| Drill | Induced fault | Alert fired at | Detection delay | Ticket |
|---|---|---|---|---|
| Service down | Stopped Nginx on srv-web01 | | | |
| Disk pressure | Controlled fill on `TODO` | | | |

Full cycle exercised: fault induced → alert raised → acknowledged with impact
assessment → service restored → alert cleared → ticket closed with cause and
preventive action.

## Verification

| Test | Expected | Result | Evidence |
|---|---|---|---|
| All intended hosts show recent data | No hosts with stale data | | |
| Alert fires on induced fault | Alert within expected window | | |
| Alert clears on recovery | Auto-resolves | | |
| Notification reaches the operator | Delivered | | |

## Known limitations

`TODO — e.g. no notification channel beyond the console? no dashboards? no
service-level checks (HTTP response body / cert expiry) beyond port checks?
single monitoring server with no HA?`
