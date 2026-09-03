# 02 · Linux baseline — server deployment and basic operations

> **TODO** — fill in the blanks marked below and drop screenshots into `evidence/`.

**Task objective.** Bring Linux servers into a known, documented state:
addressing, accounts, remote access, time synchronization, and a repeatable
health check.

## Servers

| Host | Address | Zone | Role |
|---|---|---|---|
| srv-dns01 | 192.168.40.10 | Server | Internal DNS **and** company portal (Nginx) — see note below |
| srv-mon01 | 192.168.40.20 | Server | Monitoring |
| srv-log01 | 192.168.40.30 | Server | Log collection |
| srv-auto01 | 192.168.50.10 | Ops | Automation control node |

> The plan originally placed the portal on a dedicated `srv-web01` in the DMZ
> (192.168.60.10). In the build it runs on `srv-dns01` instead — see
> `../docs/access-control-matrix.md` for why that matters.

OS: Rocky Linux 9.8 (Blue Onyx), installed via minimal ISO in VirtualBox.

VMs use two interfaces where internet access is needed during setup: a NAT
adapter (`10.0.2.0/24`, for `dnf update` and package installs) plus a static
adapter on the zone's real subnet per the addressing plan — since Packet
Tracer and VirtualBox are separate tools with no native bridge between them
(see the top-level README's Known Limitations).

## What I configured

**Static addressing and hostnames.** Each server was given a fixed address per
the addressing plan, with the internal resolver as its DNS server.

```bash
hostnamectl set-hostname srv-dns01
nmcli con show
nmcli con mod "<connection>" ipv4.addresses 192.168.40.10/24
nmcli con mod "<connection>" ipv4.gateway 192.168.40.1
nmcli con mod "<connection>" ipv4.dns 192.168.40.10
nmcli con mod "<connection>" ipv4.method manual
nmcli con up "<connection>"
```

The connection name is rarely `System eth0` — `nmcli con show` first. Changing
addressing over SSH without a console fallback is how you lose a host.

**Named administrator accounts, not a shared root.** An `ops` group with
per-person accounts, privilege via `sudo`.

```bash
groupadd -f ops
useradd -m -G wheel,ops ops01
sudo -l -U ops01
```

**SSH key authentication**, verified working before touching `sshd_config`,
with a second session held open during the change.

```bash
ssh-keygen -t ed25519 -C "ops01"
ssh-copy-id ops01@192.168.60.10
sshd -t && systemctl reload sshd
```

**Time synchronization** via chrony — a prerequisite for log correlation in
section 07, not an afterthought.

## Health check

```bash
hostnamectl; timedatectl; uptime; free -h; df -hT
ip address; ip route; ss -lntup
systemctl --failed
journalctl -p warning --since today
```

`TODO: what did the first run actually surface? Any failed units, disk pressure,
unexpected listeners? Record findings, not just that the commands ran.`

## Verification

| Test | Expected | Result | Evidence |
|---|---|---|---|
| Static address persists across reboot | Address, gateway, DNS unchanged | | |
| Key-based SSH as ops01 | Login succeeds without password | | |
| `sudo` as ops01 | Authorized commands succeed and are logged | | |
| Time sync | `timedatectl` reports synchronized | | |
| Health check | Runs clean, or findings recorded | | |

## Known limitations

`TODO — e.g. password authentication still enabled? no centralized account
management? no configuration management for the baseline itself (that arrives in
section 09)?`
