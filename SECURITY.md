# Security policy

This repository documents an isolated educational lab. The examples are not a hardened production deployment.

## Secrets and publication

Never commit passwords, Fleet service tokens, enrollment tokens, API keys, browser cookies, encryption keys, private keys, full agent diagnostics, or unredacted configuration exports. All secret values here are placeholders. The private RFC1918 addresses and lab hostnames are intentional architecture documentation.

Secrets appeared during the original setup conversation. Treat them as exposed: revoke Fleet enrollment and service tokens, invalidate affected API keys, rotate passwords and encryption keys as appropriate, and update local services. Changing Kibana encryption keys can make existing encrypted saved objects or sessions unusable; follow vendor migration guidance and keep a secure backup. This repository does not perform or claim those rotations.

If a secret reaches GitHub:
1. Revoke or rotate it immediately; deleting a file does not invalidate the credential.
2. Replace it with a placeholder and inspect all branches, tags, releases and artifacts.
3. Remove it from Git history using GitHub's sensitive-data removal guidance; account for clones and forks.
4. Confirm the old secret no longer works.
5. Review logs for unexpected access.

Do not post a secret in a public issue. Report documentation problems through an issue with sanitized details.

## Network and transport

The recorded lab uses Elasticsearch HTTP on :9200, local Kibana HTTP on :5601 and self-signed Fleet Server HTTPS on :8220. HTTP lacks confidentiality, including for authentication traffic. The `--insecure` agent flag disables Fleet certificate validation; use it only to reproduce this isolated setup.

The example binds Elasticsearch and Fleet to the Mac's lab bridge and Kibana to loopback. Docker Desktop binding and VM reachability depend on the host network; validate the actual exposure and enforce host firewall restrictions for DC01. Do not use wildcard bindings or router port forwarding.

A UTM shared network can provide outbound connectivity and is not automatically an isolated security boundary. Disconnect unnecessary routes, avoid overlap with real networks, use dedicated accounts, and never reuse production passwords. Trusted TLS, least privilege, reviewed firewall rules and supported patched versions are required before considering any wider deployment.

## Screenshot handling

Use lab accounts. Crop unnecessary content and apply solid, irreversible redaction to secrets. Review the exported image for terminals, address bars, enrollment commands, account menus, browser tabs and notifications. Never upload originals containing secrets. See [the evidence checklist](screenshots/README.md).

## Authorized testing

Only generate authentication failures on systems you own or have explicit permission to test. Check lockout policy before testing, stop after the planned attempts, and keep VM snapshots and credentials outside this repository.

References: [Fleet secure connections](https://www.elastic.co/docs/reference/fleet/secure-connections), [GitHub sensitive-data removal](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository).
