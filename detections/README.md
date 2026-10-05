# Detection engineering

Current detection: [Multiple Failed Kerberos Logins](multiple-failed-kerberos-logins.md).

The lab workflow is to generate authorized authentication activity, verify the raw DC audit record, verify ingestion, test KQL in Discover, create a threshold rule, and validate the alert against the source events.

The current implementation uses raw `message` content rather than parsed ECS user/source fields. It is a documented UI rule, not an importable rule export. Preserve that scope when describing it in a portfolio. Future rules should include data dependencies, exact query, grouping/window, false positives and a repeatable validation procedure.
