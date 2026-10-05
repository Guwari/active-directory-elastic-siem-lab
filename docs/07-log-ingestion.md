# 07 — Log ingestion

## Configure Custom Logs

Within **DC01 Linux Policy**, add the **Custom Logs (Filestream)** integration:

| Setting | Value |
|---|---|
| Integration name | Samba authentication logs (rebuild suggestion) |
| Log path | /var/log/samba/log.samba |
| Dataset | samba.auth |
| Namespace | default (rebuild suggestion) |
| Agent target | DC01 Linux Policy |

Package/UI labels can vary. The dataset and file path are the recorded values. Save and confirm DC01 receives the new policy revision.

The baseline preserves raw `message` text. It does not promise ECS usernames, source addresses, mechanism fields or parsed Samba timestamps. If headers/status are split across lines, inspect the records before configuring multiline rules; do not invent a parser without matching real samples.

## Verify source and permissions

```bash
sudo testparm -s
sudo tail -n 50 /var/log/samba/log.samba
sudo stat /var/log/samba/log.samba
sudo elastic-agent status
```

Generate a fresh domain authentication event. Confirm the Agent process can read the file without loosening it to world-readable. Review log rotation and verify collection continues after a normal rotation.

## Verify in Discover

Open Discover, select/create a `logs-*` data view with `@timestamp`, and use Last 15 minutes:

```kql
data_stream.dataset : "samba.auth"
```

Then filter failures:

```kql
data_stream.dataset : "samba.auth" and
message : "NT_STATUS_WRONG_PASSWORD"
```

Inspect `@timestamp`, `message`, `host.name`, `log.file.path`, `data_stream.dataset`, and the actual data stream/index name. Expected origin: DC01 and `/var/log/samba/log.samba`. A rebuild with default namespace should normally use `logs-samba.auth-default`; verify rather than assume.

Raw Filestream may timestamp collection rather than the original Samba occurrence unless an ingest pipeline parses it. Rule scheduling uses the indexed timestamp; time drift and ingestion delays matter.

## Acceptance

- A new DC audit record appears in Discover.
- Dataset equals `samba.auth`.
- A controlled wrong-password test produces the expected status.
- File path and host identify the intended source.
- The same messages are not duplicated by multiple input configurations.

A Healthy Fleet agent is necessary but does not itself prove log ingestion. Some Filestream fingerprint settings wait for enough bytes before ingesting a tiny file; inspect the integration's fingerprint/minimum-size settings if a new small file appears idle.

Next: [08 — Detection engineering](08-detection-engineering.md).
