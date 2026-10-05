# 01 — Lab overview

## Objective

Build an enterprise-style identity lab and centralize DC authentication telemetry for investigation and detection engineering. The completed baseline covers domain authentication, Samba auditing, SIEM ingestion and a custom threshold rule.

## Recorded environment

| Item | Recorded value |
|---|---|
| Host | Apple Silicon Mac; UTM and Docker Desktop |
| Network | UTM shared network, 192.168.64.0/24 |
| DC | DC01, Debian Samba AD, 192.168.64.12 |
| Endpoint | WIN11-LAB02, Windows 11, 192.168.64.10 |
| Realm / NetBIOS | LAB.LOCAL / LAB |
| Mac bridge | bridge100, 192.168.64.1 |
| Elasticsearch / Kibana | 9.2.0 / 9.2.0 |
| Audit file / dataset | /var/log/samba/log.samba / samba.auth |

Debian, Samba and Windows build numbers, UTM release and integration package versions were not retained. The commands are a rebuild guide for fresh disposable guests. Record `cat /etc/os-release`, `samba --version`, `uname -m`, Windows `winver`, Docker versions and the Custom Logs package version in your own evidence.

## Prerequisites

- UTM with a Debian guest and domain-join-capable Windows 11 edition (Pro, Enterprise or Education).
- Enough host memory for both guests and Docker. The sample Elasticsearch heap is 2 GB; provide additional container memory and adjust guest resources to your host.
- Local administrator access, internet access for official packages, and lab-only credentials.
- Available static addresses, synchronized clocks, and a subnet that does not overlap real networks.
- Read [SECURITY.md](../SECURITY.md), snapshot fresh guests, and rotate any previously exposed credentials.

## Build order and checkpoints

| Stage | Success criterion |
|---|---|
| Network | Windows resolves DC01 through DC01 DNS |
| Domain | Windows joins LAB.LOCAL and a test user signs in |
| Audit | DC01 log contains authentication outcomes |
| Elastic | Authenticated Elasticsearch responds; Kibana loads |
| Fleet | DC01 agent is Healthy and receives its policy |
| Ingestion | Discover shows samba.auth and wrong-password messages |
| Detection | Rule is created and enabled with intended settings |
| Validation | A matching alert is linked to source events |

Only the first seven stages were confirmed in the retained lab conversation. Do not mark alert validation complete until you capture it.

Next: [02 — Network architecture](02-network-architecture.md).
