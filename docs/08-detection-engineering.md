# 08 — Detection engineering

First verify [Discover results](07-log-ingestion.md). In Kibana → Security → Rules → Detection rules, create a **Threshold** rule. Menu labels can vary; use Kibana search if needed.

## Configure

- Name: **LAB - Multiple Failed Kerberos Logins**
- Description: Detects repeated failed-password events in Samba DC authentication logs; Kerberos is the lab test scenario.
- Index source: use the verified Samba data stream, such as `logs-samba.auth-*`.
- Language: KQL.

```kql
data_stream.dataset : "samba.auth" and
message : "NT_STATUS_WRONG_PASSWORD"
```

Set threshold **3**, leave Group by empty, run every **1 minute**, and set additional look-back **1 minute**. Do not group by absent/unparsed username or source fields. No distinct-value cardinality condition is required.

Severity/risk were not retained in the original walkthrough. For a new lab rule, start with low / 21 if required, document that choice, and tune based on validation. Leave external notification actions unconfigured unless intentionally added.

Create and enable the rule. Confirm successful execution in its execution log/monitoring page. Rule execution needs an authorized API key and stable Kibana encrypted saved-object configuration; recreate/refresh the key under an appropriate user if privileges change.

## Interpret correctly

The query detects three matching documents across the dataset in the evaluated window. It does not constrain authentication mechanism or identity. The one-minute interval plus one-minute extra look-back covers approximately two minutes per run; this is not a fixed three-attempt session boundary.

A threshold alert summarizes aggregated matches. To recover raw authentication context, inspect the underlying data in Discover.

Full specification: [Multiple Failed Kerberos Logins](../detections/multiple-failed-kerberos-logins.md). Rule creation was confirmed in the original lab; alert firing remains a separate acceptance check.

Reference: [Elastic rule creation and threshold behavior](https://www.elastic.co/docs/solutions/security/detect-and-alert/using-the-rule-ui).

Next: [09 — Testing and validation](09-testing-and-validation.md).
