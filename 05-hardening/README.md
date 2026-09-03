# 05 · Host hardening — Linux security baseline

> **TODO** — fill in the blanks marked below and drop screenshots into `evidence/`.

**Task objective.** Establish a Linux baseline that is checkable, verifiable and
reversible — covering accounts, authentication, exposed services, file
permissions and audit logging.

Every item below is stated as *check → finding → change → re-check*. A hardening
list without the "finding" column is a wish list.

## Accounts and authentication

```bash
awk -F: '($3 == 0) {print $1}' /etc/passwd      # UID 0 accounts
awk -F: '($2 == "") {print $1}' /etc/shadow     # empty passwords
passwd -S ops01
last -a | head; lastb -a | head
```

| Check | Finding | Change made | Re-check result |
|---|---|---|---|
| Accounts with UID 0 | | | |
| Empty or non-expiring passwords | | | |
| Accounts with no owner | | | |
| Password complexity and lockout policy | | | |

## SSH

```bash
grep -E '^(PermitRootLogin|PasswordAuthentication|MaxAuthTries|AllowGroups)' /etc/ssh/sshd_config
sshd -t && systemctl reload sshd
```

Applied: `PermitRootLogin no`, `MaxAuthTries 4`, `AllowGroups ops`.
`TODO: was PasswordAuthentication disabled? If not, say why — "keys were not
distributed to all admins yet" is a legitimate answer and shows you weighed it.`

## Services and exposed ports

```bash
systemctl list-unit-files --state=enabled
ss -lntup
firewall-cmd --list-all
```

Every listening port reconciled against the asset inventory.

| Port | Service | Business justification | Action |
|---|---|---|---|
| | | | |

`TODO: fill from your actual ss output. Anything you could not justify and
disabled is the most interesting row here.`

## File permissions and audit

```bash
find / -xdev -perm -4000 -type f 2>/dev/null    # SUID binaries
find /etc -xdev -type f -perm /022 -ls          # world-writable config
stat /etc/passwd /etc/shadow /etc/ssh/sshd_config
systemctl enable --now auditd
ausearch -m USER_LOGIN --start today
```

Audit coverage: account changes, authentication failures, modification of key
configuration files, privileged command use.

## Verification

| Check | Method | Expected | Result | Evidence |
|---|---|---|---|---|
| Root SSH login | Attempt `ssh root@host` | Rejected | | |
| Admin account | Key login + `sudo` | Succeeds, logged | | |
| Exposed ports | `ss` + external port test | Approved ports only | | |
| Audit trail | Make a change, then `ausearch` | Change is retrievable | | |
| Service availability | Portal, DNS, monitoring | Still working after hardening | | |

That last row matters most. Hardening that breaks the service is not hardening.

## Known limitations

`TODO — e.g. no SELinux policy work? no automated baseline drift detection? no
CIS benchmark mapping? baseline applied by hand rather than by section 09?`
