# 07 · Logging and incident response — Wazuh

> **TODO** — fill in the blanks marked below and drop screenshots into `evidence/`.

**Task objective.** Collect host, authentication, web and network security logs
centrally, and turn alerts into defensible findings rather than guesses.

Platform: Wazuh `TODO: version` on srv-log01 (192.168.40.30)

## Prerequisite

Time synchronization across all sources. Without it, correlating an
authentication failure on one host with a firewall log on another is guesswork.
Section 01 notes that the switches currently have no NTP and no syslog
forwarding — so device events are not in this platform yet.

## Log sources

| Source | Host | Path / method | Status |
|---|---|---|---|
| System log | All Linux | `/var/log/messages` via agent | |
| Authentication | All Linux | `/var/log/secure` via agent | |
| Audit | All Linux | auditd via agent | |
| Nginx access | srv-web01 | `/var/log/nginx/portal_access.log` | |
| Nginx error | srv-web01 | `/var/log/nginx/portal_error.log` | |
| Firewall | Edge | `TODO` | |
| Network devices | Switches | Not yet forwarded — see section 01 | ❌ |

```bash
systemctl enable --now wazuh-agent
tail -n 50 /var/ossec/logs/ossec.log
```

## Investigation method

For each alert: confirm time, rule, source, target and account → go back to the
raw log → determine whether it was a test, a mistake, or genuinely hostile →
scope it (same source, same account, other hosts?) → grade it on evidence →
contain → verify the risk is closed → record it.

The discipline that matters: **do not write "intrusion" without evidence.** Most
authentication-failure alerts in a lab are the operator, and saying so is the
correct finding.

## Investigations

| ID | Alert | Raw evidence checked | Verdict | Action | Verified |
|---|---|---|---|---|---|
| SEC-01 | `TODO e.g. repeated SSH authentication failures` | | | | |
| SEC-02 | | | | | |

`TODO: write up at least one investigation in full — what fired, what you looked
at, what you concluded and why, what you did, how you confirmed it worked. One
complete investigation is worth more than ten alert screenshots. If the verdict
was "this was me running a test", that is a legitimate and honest write-up.`

## Verification

| Test | Expected | Result | Evidence |
|---|---|---|---|
| All agents connected | No disconnected agents | | |
| Induced auth failure appears | Alert within expected window | | |
| Nginx 4xx/5xx appears | Searchable in the platform | | |
| Historical search by host and time | Returns matching events | | |

## Known limitations

`TODO — e.g. network device logs not forwarded? no log retention policy? no
alerting/notification on high-severity rules? no file integrity monitoring
tuning?`
