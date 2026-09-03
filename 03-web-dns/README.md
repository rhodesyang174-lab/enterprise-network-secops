# 03 · Web & DNS — Nginx portal and internal resolver

> **TODO** — a few blanks below still need your input; drop screenshots into `evidence/`.

**Task objective.** Bring the company portal and internal name resolution online,
with configuration validation, backup, and a tested restore path.

> **Deployment note.** The original plan put the portal on a dedicated host in
> the DMZ (`srv-web01`, 192.168.60.10). In the build, Nginx runs alongside
> internal DNS on **the same host**, `srv-dns01` (192.168.40.10, server zone).
> VLAN 60 has no host deployed. The security consequence of that is covered in
> [`../docs/access-control-matrix.md`](../docs/access-control-matrix.md) and
> the top-level README's Known Limitations — this section covers the service
> configuration itself.

## Nginx portal — srv-dns01 (192.168.40.10)

Confirmed running as a systemd service, config validated before every reload:

```
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: disabled)
   Active: active (running)
```

```bash
nginx -t
# nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
# nginx: configuration file /etc/nginx/nginx.conf test is successful
systemctl reload nginx
curl -I http://127.0.0.1
# HTTP/1.1 200 OK
# Server: nginx
# Content-Type: text/html
```

`nginx -t` before every reload — a syntax error caught at reload time is an
outage; caught at test time it is a typo. This was exercised for real during
the section 10 incident drills (INC-03): the service was stopped deliberately,
`nginx -t` confirmed the config itself was still valid, and the fix was a
restart, not a config change.

`TODO: paste the actual server {} block from /etc/nginx/nginx.conf (or the
conf.d file) and note the document root and log paths you used.`

## Internal DNS — srv-dns01 (192.168.40.10)

BIND (`named`), confirmed listening on the server's addresses on port 53:

```
tcp   LISTEN   0   10   192.168.40.10:53
tcp   LISTEN   0   10   127.0.0.1:53
```

```bash
named-checkconf
named-checkzone example.lab /var/named/example.lab.zone
```

`TODO: paste the actual zone file content (the SOA/NS/A records you defined).`

> **Open item — DNS answer needs re-checking.** During the INC-02 recovery
> verification in section 10, `dig @127.0.0.1 portal.example.lab` returned
> `status: NOERROR` but **zero records in the answer section**, with a root
> server SOA in the authority section — normally a sign the query is being
> answered recursively rather than from a locally authoritative zone, i.e.
> `example.lab` may not be loading as an authoritative zone even though
> `named` itself is running. Recording this honestly rather than glossing
> over it: **`systemctl is-active named` returning "active" is not the same
> as the zone actually resolving.** `TODO: re-run `dig @192.168.40.10
> portal.example.lab` after checking `named.conf` zone stanza and file
> path/permissions, and update this section with the real result.`

> **Naming inconsistency to resolve:** the switches use `ip domain-name xyang.lab`
> while DNS serves `example.lab`. `TODO: pick one and align both.`

## Backup and restore

```bash
tar -czf /backup/config/nginx-$(date +%F).tar.gz /etc/nginx /var/www/company
tar -czf /backup/config/dns-$(date +%F).tar.gz /etc/named.conf /var/named
sha256sum /backup/config/*.tar.gz > /backup/config/SHA256SUMS
```

Restore procedure followed: extract to a temporary path first, inspect contents
and permissions, back up the current state, then replace — never extract
straight over a live directory.

**Restore drill result:** `TODO — did you actually restore from backup and
confirm the service came back? A backup that has never been restored is a
hypothesis. This is one of the most valuable things in the whole repo if you
did it; say what broke, if anything.`

## Verification

| Test | Expected | Result | Evidence |
|---|---|---|---|
| `nginx -t` before reload | Syntax OK | Pass | Confirmed during INC-03 drill, section 10 |
| Portal reachable locally | `curl -I http://127.0.0.1` → HTTP 200 | Pass | Confirmed during INC-03 recovery, section 10 |
| Portal reachable from srv-mon01 | `curl -I http://192.168.40.10` → HTTP 200 | Pass | Confirmed during INC-03 drill, section 10 |
| `named` service active | `systemctl is-active named` | Pass | Confirmed during INC-02 drill, section 10 |
| `portal.example.lab` resolves to a real answer | Non-empty ANSWER section | **Open — see note above** | |
| Resolver refuses queries from outside allowed ranges | Query refused | `TODO` | |
| Restore from backup | Service returns, config validates | `TODO` | |

## Known limitations

1. **Portal and DNS share a host and a zone**, with no network boundary
   between them — see the deployment note above and
   `../docs/access-control-matrix.md`.
2. **DNS zone resolution needs re-verification** — service is active but the
   actual answer to a query for the portal record was empty in the last
   recorded test. See the open item above.
3. `TODO — e.g. HTTP only, no TLS? single resolver with no secondary? no zone
   transfer restrictions? backups stored on the same host they protect? has a
   restore drill actually been run?`
