# 09 — Testing and validation

The retained conversation establishes working domain authentication, Healthy DC01 enrollment, searchable Samba events, wrong-password search results, and rule creation. It does not contain an explicit confirmation of a generated Security alert. Complete this procedure to close that evidence gap.

## Prepare

1. Confirm all services, DC01 Agent and the rule are healthy/enabled.
2. Use a disposable `LAB\\testuser` account and inspect `samba-tool domain passwordsettings show` before testing.
3. Synchronize clocks, record the start in UTC, and avoid unrelated failure traffic.
4. Confirm a legitimate domain sign-in and DC audit event; cached workstation credentials alone are insufficient.
5. Observe a quiet window with fewer than three matching documents.

## Positive test

On WIN11-LAB02, lock the workstation and attempt domain sign-in using an intentionally wrong password, up to three times. Stop after those attempts and do not automate or extend guessing. If lockout policy would trigger earlier, adjust the test safely in this disposable lab.

On DC01, verify freshly generated records containing `NT_STATUS_WRONG_PASSWORD`. In Discover, run the documented KQL and inspect the raw mechanism, user, source and timestamps. Count actual events; Windows retries can produce multiple events per attempt.

Allow ingestion and at least two scheduled rule executions. Open Security → Alerts, filter the exact rule name, and inspect the alert details and execution history. If no alert appears, check [troubleshooting](10-troubleshooting.md); do not assert success.

## Negative check

In a later quiet window, generate fewer than three matching documents and verify no new threshold alert attributable to that window. Count documents and consider overlapping windows before drawing a conclusion. A previous positive test may still be within the look-back period.

## Evidence record

| Check | Expected | Record locally |
|---|---|---|
| Domain identity | LAB test user and live DC contact | UTC, host/domain screenshot |
| Source | DC01 log has failure status | UTC, sanitized record |
| Ingestion | samba.auth documents visible | UTC, count, source path |
| Rule | Query, threshold 3, no grouping, 1m + 1m | Sanitized settings screenshot |
| Execution | Successful scheduled run | UTC and execution result |
| Alert | Exact rule name and matching count | UTC, sanitized alert ID/count |
| Negative window | No new alert for fewer than 3 matches | UTC window and event count |

Use [screenshot guidance](../screenshots/README.md). Do not upload credentials, tokens, raw diagnostics or unrelated personal details. An alert screenshot should preserve the rule name, timestamp and threshold count while redacting sensitive material.

Acceptance is an evidenced source → ingestion → rule → alert chain. Do not describe attack detection effectiveness or performance benchmarks from this small test.

Next: [10 — Troubleshooting](10-troubleshooting.md).
