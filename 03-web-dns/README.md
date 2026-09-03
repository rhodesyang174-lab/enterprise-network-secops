# 03 · Web & DNS — Nginx portal and internal resolver

> **TODO** — fill in the blanks marked below and drop screenshots into `evidence/`.

**Task objective.** Bring the company portal and internal name resolution online,
with configuration validation, backup, and a tested restore path.

## Nginx portal — srv-web01 (192.168.60.10, DMZ)

```nginx
server {
    listen 80;
    server_name portal.example.lab;
    root /var/www/company;
    index index.html;
    access_log /var/log/nginx/portal_access.log;
    error_log  /var/log/nginx/portal_error.log warn;
}
```

Content directory owned `root:nginx` at mode 750 — the worker process reads it,
nothing writes to it.

```bash
nginx -t && systemctl reload nginx
curl -I http://127.0.0.1
```

`nginx -t` before every reload. A syntax error caught at reload time is an
outage; caught at test time it is a typo.

## Internal DNS — srv-dns01 (192.168.40.10, Server zone)

BIND, with listening addresses and `allow-query` restricted to internal ranges
rather than left at defaults.

```dns
$TTL 86400
@  IN SOA srv-dns01.example.lab. admin.example.lab. (
       2026082401 3600 900 604800 86400 )
   IN NS  srv-dns01.example.lab.
srv-dns01 IN A 192.168.40.10
portal    IN A 192.168.60.10
```

```bash
named-checkconf
named-checkzone example.lab /var/named/example.lab.zone
dig @192.168.40.10 portal.example.lab
```

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
| Portal reachable from an office workstation | HTTP 200 | | |
| `dig portal.example.lab` from an office VLAN | Resolves to 192.168.60.10 | | |
| Resolver refuses queries from outside allowed ranges | Query refused | | |
| Restore from backup | Service returns, config validates | | |

## Known limitations

`TODO — e.g. HTTP only, no TLS? single resolver with no secondary? no zone
transfer restrictions? backups stored on the same host they protect?`
