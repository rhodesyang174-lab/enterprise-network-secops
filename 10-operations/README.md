# 10 · Operations — health checks, change control, incident handling

> **TODO** — fill in the blanks marked below and drop screenshots into `evidence/`.

**Task objective.** Put a repeatable process around running the environment, so
that work can be handed over, audited and learned from.

This section is the one most homelab projects skip, and the one that most
resembles an actual operations job.

## Routine health check

| Target | Checks | Abnormal when |
|---|---|---|
| Network devices | CPU, memory, interfaces, error counters, routes, logs | Sustained resource pressure, flapping interfaces, missing routes |
| Firewall | Interfaces, sessions, rule hit counts, system log, config backup | Session anomalies, dead rules, config not backed up |
| Linux servers | Load, memory, disk, processes, ports, failed units | Threshold breach, failed service, unexpected listener |
| Nginx / DNS | Process, port, config test, error log, functional request | Service unavailable, clustered errors |
| Monitoring | Data collection, alerts, notification path, time sync | Host with no data, alert not delivered, clock drift |
| Backups | Job status, file size, checksum, spot restore | Job failed, file anomaly, restore fails |

## Incident handling flow

Intake → reproduce and scope → check recent changes → gather monitoring, logs,
config and connectivity evidence → narrow layer by layer (endpoint → access →
core → edge → server → application) → contain and restore → verify from the
user, network and service sides → record root cause and preventive action.

**Restore first, root-cause second.** Users care about the service being back;
the analysis can follow.

## Incidents handled

Drawn from the fault library in the project plan. Fill in the ones you actually
worked through.

| ID | Symptom | Root cause | Key evidence | Write-up |
|---|---|---|---|---|
| INC-01 | R&D cannot reach the server zone | VLAN missing from trunk allowed list | `show interfaces trunk`, port status | |
| INC-02 | Name resolution fails | Zone record or client resolver wrong | `dig`, zone file, named log | |
| INC-03 | Portal unreachable | Nginx stopped or firewall denying | `systemctl`, `ss`, `curl`, policy log | |
| INC-04 | Portal noticeably slow | CPU contention | Monitoring graph, `top` | |
| INC-05 | Disk usage alert | Logs not rotating | `df`, `du`, logrotate config | |
| INC-06 | Burst of SSH auth failures | Repeated failed logins from a test account | Auth log, source address, Wazuh alert | |
| INC-07 | Administrator cannot log in | Firewall rule ordering | Rule hit counts, port test, auth log | |
| INC-08 | Web content modified unexpectedly | Permissions too broad | File attributes, audit log, change record | |
| INC-09 | Ansible run fails | Inventory or key problem | Play output, SSH test, inventory | |
| INC-10 | Config lost after device reboot | Running config never saved | Startup config, change record | |

`TODO: write up two of these in full in this folder — symptom, what you checked
and in what order, what you ruled out, root cause, fix, and how you verified.
The ones where your first hypothesis was wrong are the best ones to pick; being
able to describe a wrong turn is what separates a real troubleshooting story
from a rehearsed one.`

## Templates

Blank forms used during the build: [`templates/`](templates/)

- Incident ticket
- Change record
- Security event record
- Acceptance checklist

## Known limitations

`TODO — e.g. tickets tracked in documents rather than a ticketing system? health
checks run manually rather than scheduled? no defined SLA or escalation path?`
