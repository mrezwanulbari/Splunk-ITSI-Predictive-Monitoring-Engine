# Entity Definition: Web Server Fleet

Example entity type definition showing how to model a fleet of web servers as ITSI entities with the identifying fields needed for correct KPI-to-entity correlation.

## Entity Type Fields

| Field | Purpose |
|---|---|
| `host` | Primary identifier — must match the `host` field exactly as it appears in the underlying index data |
| `dns` | Alias field for correlation when logs reference the DNS name instead of hostname |
| `ip` | Alias field for correlation when logs reference IP address (common in load balancer / firewall logs) |
| `environment` | Info field — tags entity as prod/staging/dev, used for service filtering |
| `role` | Info field — e.g. "web-frontend", used for grouping in glass tables |

## Import Method

For fleets larger than a handful of hosts, define entities via a scheduled search against your CMDB or asset inventory rather than manual entry — manual entity lists silently go stale the moment infrastructure changes, which quietly breaks KPI-to-entity correlation without any visible error.

```spl
| inputlookup cmdb_web_servers.csv
| eval title=hostname
| fields title, hostname, ip_address, environment, role
```

Schedule this as an entity import search on a daily cadence so decommissioned hosts age out and new ones are picked up automatically.

---
*Part of the [Splunk-ITSI-Predictive-Monitoring-Engine](../README.md) repository.*
