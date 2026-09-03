# 02 · Linux baseline — server deployment and basic operations

**Task objective.** Bring Linux servers into a known, documented state:
addressing, accounts, remote access, time synchronization, and a repeatable
health check.

## Servers

| Host | Address | Zone | Role | Build status |
|---|---|---|---|---|
| srv-dns01 | 192.168.40.10 | Server | Internal DNS **and** company portal (Nginx) | Built — evidence below |
| srv-mon01 | 192.168.40.20 | Server | Monitoring | `TODO` |
| srv-log01 | 192.168.40.30 | Server | Log collection | `TODO` |
| srv-auto01 | 192.168.50.10 | Ops | Automation control node | `TODO` |

> The plan originally placed the portal on a dedicated `srv-web01` in the DMZ
> (192.168.60.10). In the build it runs on `srv-dns01` instead — see
> `../docs/access-control-matrix.md` for why that matters.

OS: **Rocky Linux 9.8 (Blue Onyx)**, kernel `5.14.0-687.10.1.el9_0.1.x86_64`,
installed via minimal ISO in VirtualBox (2 vCPU, 2048 MB RAM, 20 GB disk).

VMs use two interfaces: a NAT adapter (`10.0.2.0/24`) for internet access during
setup (`dnf update`, package installs), plus a static adapter on the zone's real
subnet per the addressing plan — since Packet Tracer and VirtualBox are
separate tools with no native bridge between them (see the top-level README's
Known Limitations).

---

## srv-dns01 — build log

### 1. Install and set hostname

```
vbox login: root
Password:
[root@vbox ~]# hostnamectl set-hostname srv-dns01
[root@vbox ~]# exec bash
[root@srv-dns01 ~]#
```

### 2. Addressing

`nmcli device status` at this point showed a single interface, `enp0s3`,
picking up an address from VirtualBox's NAT DHCP:

```
DEVICE  TYPE      STATE      CONNECTION
enp0s3  ethernet  connected  enp0s3
```

```
2: enp0s3: ... mtu 1500 ...
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
```

```
[root@srv-dns01 ~]# ip route
default via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100
10.0.2.0/24 dev enp0s3 proto kernel src 10.0.2.15 metric 100
```

`proto dhcp` on that route confirms this interface is DHCP-assigned — correct
for the NAT/internet-access adapter. The second, static interface at
192.168.40.10 (the one that matters for the addressing plan) is a separate
adapter; **`TODO`: capture `nmcli device status` and `ip addr show` together,
showing both interfaces at once, so the static 192.168.40.10 assignment is
actually evidenced rather than asserted.**

### 3. Time synchronization

```bash
dnf install chrony -y
systemctl enable --now chronyd
chronyc sources
timedatectl
```

```
System clock synchronized: yes
NTP service: active
RTC in local TZ: no
```

`chronyc sources` showed four active NTP sources reachable and being polled
(`^*` marking the selected source). Confirmed synchronized before anything
else was configured — this matters because sections 06/07/10 all rely on
timestamps lining up across hosts.

### 4. System update

```bash
dnf update -y
```

Completed cleanly — kernel, NetworkManager, chrony, and related packages
upgraded, ending on `kernel-5.14.0-687.42.1.el9_0.1`. No errors.

### 5. Firewall

```bash
systemctl enable --now firewalld
firewall-cmd --list-all
```

```
public (active)
  target: default
  interfaces: enp0s3
  services: cockpit dhcpv6-client ssh
  ports:
  masquerade: no
```

Default zone active on boot, only `cockpit`, `dhcpv6-client` and `ssh` open at
this stage — before Nginx/DNS were added (those show up as open ports once
section 03 is reflected here; `TODO: re-run `firewall-cmd --list-all` now that
http/dns services are running and confirm the zone reflects them).

### 6. Account creation

```bash
id yang01
```

```
uid=1000(yang01) gid=1000(yang01) groups=1000(yang01),10(wheel)
```

Account is in the `wheel` group, which grants `sudo` on Rocky by default.
**`TODO`: this hasn't been confirmed directly — run `sudo -l -U yang01` and
capture the output.** Also worth reconciling: the section 10 incident drills
reference an account called `ops01` for SSH access, but the account actually
created here is `yang01`. `TODO: confirm which account name is the real one in
use and make this consistent across sections 02 and 10 — either rename, or
note that both exist and why.`

### 7. Basic system inventory

```bash
hostname            # srv-dns01
uname -r             # 5.14.0-687.10.1.el9_0.1.x86_64
cat /etc/rocky-release   # Rocky Linux release 9.8 (Blue Onyx)
free -h
df -h
```

```
              total   used   free   shared  buff/cache  available
Mem:          1.7Gi   483Mi  636Mi  3.0Mi   836Mi       1.3Gi
Swap:         2.0Gi   1.0Mi  2.0Gi

Filesystem              Size  Used Avail Use% Mounted on
/dev/mapper/rlm_vbox-root 17G  2.3G   14G  14% /
/dev/sda1                960M  458M  503M  48% /boot
```

Nothing concerning — plenty of free memory and disk headroom at this point in
the build.

Evidence: `evidence/srv-dns01-install.jpg`, `evidence/srv-dns01-network.jpg`,
`evidence/srv-dns01-update-firewall-users.jpg`

---

## srv-mon01 / srv-log01 / srv-auto01 — build log

`TODO` — same process as srv-dns01 above (install → hostname → addressing →
time sync → update → firewall → account). Doesn't need to be documented in
this much detail for each host; a `hostnamectl` + `ip addr show` per host is
enough to show each one was actually built, plus anything that differed from
the srv-dns01 process.

---

## SSH key authentication

**Not yet evidenced.** Planned approach:

```bash
# from the admin workstation
ssh-keygen -t ed25519 -C "yang01-lab"
ssh-copy-id yang01@192.168.40.10
ssh yang01@192.168.40.10
```

```bash
# on the server, after confirming key login works
sudo sshd -t && sudo systemctl reload sshd
```

`TODO: capture the login succeeding (prompt changes to the server's), and
sshd_config afterward if PasswordAuthentication was disabled.`

## Health check

```bash
hostnamectl; timedatectl; uptime; free -h; df -hT
ip address; ip route; ss -lntup
systemctl --failed
journalctl -p warning --since today
```

The `daily_check.sh` script built in section 10 already exercises most of
this in one pass — its output against `srv-dns01` showed `0 loaded units
listed` under `systemctl --failed` (clean) and Nginx/DNS both active. See
[`../10-operations/README.md`](../10-operations/README.md) for the full
output rather than duplicating it here.

## Verification

| Test | Expected | Result | Evidence |
|---|---|---|---|
| Rocky Linux installs and boots | Reaches login prompt, hostname set | Pass | `evidence/srv-dns01-install.jpg` |
| Time sync | `timedatectl` reports synchronized | Pass | `evidence/srv-dns01-network.jpg` |
| System update completes | No errors | Pass | `evidence/srv-dns01-update-firewall-users.jpg` |
| Firewall active on boot | `firewalld` zone active | Pass | `evidence/srv-dns01-update-firewall-users.jpg` |
| Account created, in `wheel` | `id` shows correct groups | Pass | `evidence/srv-dns01-update-firewall-users.jpg` |
| Static address on the zone subnet | 192.168.40.10 assigned | `TODO` | |
| `sudo` actually works for the account | Authorized commands succeed | `TODO` | |
| Key-based SSH | Login succeeds without password | `TODO` | |
| Other three servers built | Hostname + address confirmed each | `TODO` | |

## Known limitations

1. **Static addressing to 192.168.40.10 is not directly evidenced yet** —
   only the NAT/DHCP interface (10.0.2.15) has been captured. The static
   adapter needs its own `nmcli`/`ip addr` screenshot.
2. **Account naming is inconsistent** — `yang01` was created here; `ops01` is
   referenced in section 10's incident drills. Needs reconciling.
3. **`sudo` privilege is inferred from group membership, not directly tested**
   — `sudo -l` hasn't been run.
4. **SSH key authentication is planned but not yet evidenced.**
5. **srv-mon01, srv-log01, srv-auto01 are not yet built or documented here** —
   only srv-dns01 has a build log.
6. No centralized account management (that's a section 09 concern) and no
   configuration management applied to this baseline itself yet.
