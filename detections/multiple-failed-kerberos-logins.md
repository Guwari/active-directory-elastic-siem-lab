# LAB - Multiple Failed Kerberos Logins

## Purpose and source

Identify repeated password failures in Samba DC authentication telemetry. The lab scenario uses Kerberos authentication from WIN11-LAB02, but the implemented query is protocol-agnostic.

Source: DC01 `/var/log/samba/log.samba`, ingested by Custom Logs (Filestream) as `samba.auth`.

```kql
data_stream.dataset : "samba.auth" and
message : "NT_STATUS_WRONG_PASSWORD"
```

## Implemented settings

| Setting | Value |
|---|---|
| Rule type | Threshold |
| Threshold | 3 matching events (at least 3) |
| Group by | Empty / none |
| Interval | 1 minute |
| Additional look-back | 1 minute |
| Language | KQL |
| Index source | Select actual Samba data stream; rebuild example `logs-samba.auth-*` |

The source index was not captured in the retained rule walkthrough. Verify it in Discover and use the matching pattern. Severity/risk settings were also not recorded; a reasonable rebuild starting point is low severity / risk score 21, to be tuned, not presented as the historical value.

The interval plus additional look-back typically searches roughly the previous two minutes per run. Confirm scheduling and timestamp behavior in rule execution details. Overlapping windows and repeated tests can affect alert counts.

## Important limitations

- Ungrouped threshold counts all matching events across the selected dataset. It does not establish that a single user or source generated all failures.
- A password typo, stale credential, service configuration, or repeated protocol retries can match.
- Three log events do not guarantee three distinct password guesses.
- The query does not require `Kerberos`; inspect each raw event to establish mechanism.
- Raw Filestream events may use collection time unless a pipeline parses the original Samba timestamp.
- Threshold alerts are aggregate records. Use Discover to inspect underlying messages rather than expecting complete usernames and source addresses on the alert.

## Test and acceptance

Use a dedicated lab account, check lockout policy, and record a quiet baseline. Perform up to three deliberate wrong-password attempts from WIN11-LAB02, then stop. Verify the events on DC01 and in Discover. Check for at least three matching documents in one evaluation window, successful rule execution, and an alert with the exact rule name. Record UTC timestamps and counts.

A negative check uses a quiet window with fewer than three matching documents and verifies no new alert attributable to that window. Because Windows can issue retries, count telemetry rather than assuming attempt counts.

The original walkthrough confirmed rule creation, not a fired alert. Follow [the full evidence procedure](../docs/09-testing-and-validation.md) to prove the final stage.

## Triage

1. Open the alert and record rule/window/count.
2. Search the dataset in that time range; inspect raw `message`, `host.name` and `log.file.path`.
3. Establish source, user, domain and authentication mechanism from the original records.
4. Compare failures with legitimate successful logins, lockouts and expected test activity.
5. Treat unexplained repeated failures as investigation leads; do not assume malicious intent.

Future tuning: parse ECS `user.name`, `source.ip`, `event.outcome` and mechanism; validate mappings; group by user/source; tune window and threshold; distinguish password spraying from single-account guessing.

Reference: [Elastic threshold rule documentation](https://www.elastic.co/docs/solutions/security/detect-and-alert/using-the-rule-ui).
