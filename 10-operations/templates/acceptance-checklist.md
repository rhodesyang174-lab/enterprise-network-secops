# Acceptance checklist

| # | Item | Pass criteria | Result | Evidence |
|---|---|---|---|---|
| 1 | Topology | Matches physical connections and zone model | | |
| 2 | VLANs and addressing | No conflicts; gateways and DHCP correct | | |
| 3 | Trunk links | Allowed list complete; links up | | |
| 4 | Routing | Forward and return paths both present | | |
| 5 | Firewall policy | Matches the access matrix; deny rules log | | |
| 6 | Device management | Reachable only from the operations zone | | |
| 7 | Linux accounts | Named accounts, sudo, disable process | | |
| 8 | SSH | Root login disabled; auth events retrievable | | |
| 9 | Nginx and DNS | Config validates; service reachable | | |
| 10 | Backup and restore | Backups complete; restore spot-checked | | |
| 11 | Monitoring | Key assets covered; alert drill passed | | |
| 12 | Logging | Sources complete; events searchable | | |
| 13 | Automation | Scoped, repeatable, results traceable | | |
| 14 | Documentation | Configs, evidence, tickets and reports present | | |
