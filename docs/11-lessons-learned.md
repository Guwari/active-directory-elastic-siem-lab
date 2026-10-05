# 11 — Lessons learned

## DNS and time are identity dependencies

A domain join relies on AD-aware DNS; an IP route alone does not establish domain discovery. Kerberos also relies on compatible clocks and correct realm/hostname configuration. Checking DNS SRV records and time before credentials avoids misleading troubleshooting.

## Agent health and data ingestion are separate

Fleet Healthy confirms management connectivity. It does not prove that a file is readable, an input is correctly configured or events reach Elasticsearch. Validate the source, Agent policy, direct Elasticsearch output and Discover independently.

## Raw logs limit detection specificity

A reliable failure-status string supports a useful first threshold rule, but the current query lacks parsed user, source and mechanism fields. Its Kerberos title describes the test scenario. Grouping and protocol-specific claims require an actual parsing pipeline and validation.

## Events are not attempts

Windows authentication retries can produce multiple DC records. A threshold of three matching events therefore does not prove three deliberate guesses, one affected account, or malicious activity. Record timestamps/counts and correlate raw context during triage.

## Reproducibility needs configuration and acceptance checks

The rebuilt Compose file explicitly aligns HTTP configuration, server-user credentials and Kibana encryption keys. Fleet Server and endpoint enrollment remain separate steps with distinct credentials. A walkthrough becomes stronger when every stage has an observable checkpoint.

## Evidence should support the claim

The original lab confirmed rule creation. The portfolio should show an authentic alert only after validation, and should not manufacture missing screenshots or performance claims. Clear limitations strengthen the technical story.

## Secrets require lifecycle management

Sanitizing a repository does not revoke secrets shared elsewhere. Rotate exposed credentials locally, retain private configuration outside Git, and inspect screenshots before publishing.

## Next iteration

1. Complete and capture alert validation.
2. Parse Samba events into validated ECS fields and original event timestamps.
3. Constrain Kerberos queries and group by account/source.
4. Add Windows event, Sysmon and PowerShell telemetry.
5. Implement trusted TLS for Elasticsearch, Kibana and Fleet.
6. Export tested rule definitions and preserve versioned integration settings without secrets.

Return to [the portfolio README](../README.md).
