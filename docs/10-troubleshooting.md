# 10 — Troubleshooting

Work from the source toward the alert. Make one change at a time and retest the failed checkpoint.

| Symptom | Inspect | Corrective direction |
|---|---|---|
| Domain not found | DNS is 192.168.64.12; AD SRV records | Fix DC DNS and guest network; remove public alternate DNS |
| Kerberos fails | Hostname, realm, clock, kinit/klist | Correct realm/FQDN and synchronize time |
| Samba won't start | testparm, service journal, DNS port conflict | Preserve provisioned settings; remove conflicting services safely |
| No audit record | auth_audit:3, log file, live DC contact | Generate a live domain request; cached login may never reach DC |
| Kibana authentication error | kibana_system password and initialized volume | Reset server-user password to local .env value; do not log in as it |
| Kibana detection errors | Encryption keys, rule permissions, execution log | Use stable keys of at least 32 characters and valid authorized rule API key |
| Fleet Server unreachable | HTTPS :8220 publishing, firewall, bridge IP | Validate host binding and lab route; preserve HTTPS |
| Certificate error enrolling | Self-signed Fleet certificate | Use trusted CA/SAN; lab-only --insecure matches historical setup |
| Agent Healthy but no data | Policy revision, file access, output URL | Verify Custom Logs settings and direct :9200 reachability |
| Elasticsearch 401 | Output credentials/API key | Confirm Fleet-issued output credentials; 401 reachability is not ingestion |
| Tiny file not ingested | Filestream fingerprint minimum size | Consult installed integration settings; generate sufficient legitimate test data |
| Duplicate documents | Multiple inputs and agent state | Keep one input per file; preserve state across restarts |
| Discover empty | Data view, dataset, time range, timezone | Search logs-* and verify current UTC/time window |
| Rule produces no alert | Index source, >=3 documents in window, enabled state | Inspect rule execution and actual indexed timestamps |
| Alert lacks user/IP | Aggregate threshold alert and raw input | Investigate source events; add an ECS parsing pipeline later |

## Useful local checks

DC01:

```bash
sudo testparm -s
sudo systemctl status samba-ad-dc
sudo journalctl -u samba-ad-dc --since "15 minutes ago"
sudo tail -n 50 /var/log/samba/log.samba
sudo elastic-agent status
```

Mac, from repository root:

```bash
docker compose --env-file .env -f configs/docker-compose.example.yml --profile fleet ps
docker compose --env-file .env -f configs/docker-compose.example.yml --profile fleet logs --tail=50
```

Review outputs privately. `docker compose config` without `--quiet`, container inspection, Agent diagnostics and enrollment command output can reveal secrets. Do not paste these into public issues.

Next: [11 — Lessons learned](11-lessons-learned.md).
