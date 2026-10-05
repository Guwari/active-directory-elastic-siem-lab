# Lab architecture

![Lab architecture](architecture-diagram.svg)

## Identity and network

UTM shared network: `192.168.64.0/24`. Mac gateway / bridge100: `192.168.64.1`. DC01 is Debian at `192.168.64.12`; WIN11-LAB02 is Windows 11 at `192.168.64.10`. DNS realm is `LAB.LOCAL`, NetBIOS domain is `LAB`.

The workstation uses DC01 for DNS and authenticates domain accounts against Samba. DNS resolution and synchronized clocks are prerequisites for Kerberos. The Windows endpoint has no documented Elastic Agent deployment in this baseline; the telemetry comes from the DC.

## Connections

| Source → destination | Protocol / purpose |
|---|---|
| Windows → DC01 | AD DNS, Kerberos, LDAP, SMB and domain services |
| DC01 Agent → Fleet Server | HTTPS :8220; enrollment, policy and check-ins |
| DC01 Agent → Elasticsearch | HTTP :9200; authentication telemetry |
| Fleet Server → Elasticsearch | Internal Docker HTTP :9200; Fleet state |
| Kibana → Elasticsearch | Internal Docker HTTP :9200; search and rule execution |
| Mac browser → Kibana | Loopback HTTP :5601 |

The diagram separates telemetry from management traffic. Agent logs are not proxied by Fleet Server. Docker service names such as `elasticsearch` work inside the Docker network; DC01 must use the Mac bridge address.

## Audit-to-alert sequence

1. WIN11-LAB02 submits a domain authentication request.
2. DC01 processes it and writes authentication audit records to `/var/log/samba/log.samba`.
3. DC01 Elastic Agent tails the file using Filestream.
4. Agent publishes documents with `data_stream.dataset = samba.auth` to Elasticsearch.
5. Kibana's Security rule evaluates matching events and creates a threshold alert if its conditions are met.

The SVG is an editable repository asset with no external fonts, scripts or dependencies. It documents the recorded components and an expected detection outcome, not a screenshot proving a fired alert. See [security boundaries](../SECURITY.md).
