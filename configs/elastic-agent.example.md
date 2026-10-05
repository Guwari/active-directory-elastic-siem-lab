# DC01 Elastic Agent example

Use Fleet's generated installation instructions for the **DC01 Linux Policy** and the correct OS/CPU. For an ARM64 Debian guest, the 9.2.0 archive is `elastic-agent-9.2.0-linux-arm64.tar.gz`; x86_64 guests need their matching archive. Check `uname -m` first.

Download only from [Elastic's official artifacts](https://www.elastic.co/downloads/elastic-agent), verify the published checksum, and extract locally. For example, on ARM64:

```bash
curl -fLO https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-9.2.0-linux-arm64.tar.gz
curl -fLO https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-9.2.0-linux-arm64.tar.gz.sha512
sha512sum -c elastic-agent-9.2.0-linux-arm64.tar.gz.sha512
tar -xzf elastic-agent-9.2.0-linux-arm64.tar.gz
cd elastic-agent-9.2.0-linux-arm64
```

Enter the fresh enrollment token privately on DC01. The following example avoids putting its literal value in shell history; it still appears in process arguments during installation. Do not capture installation output for publication.

```bash
read -rsp 'DC01 policy enrollment token: ' LAB_ENROLLMENT_TOKEN; echo
sudo ./elastic-agent install \
  --url=https://192.168.64.1:8220 \
  --enrollment-token="$LAB_ENROLLMENT_TOKEN" \
  --insecure
unset LAB_ENROLLMENT_TOKEN
sudo elastic-agent status
```

The token's documentation placeholder is `<ELASTIC_AGENT_ENROLLMENT_TOKEN>`. A Fleet Server service token is a different credential and must not be substituted.

**Lab warning:** `--insecure` skips certificate verification. For trusted TLS use a CA/certificate with the Fleet hostname or IP in its SAN, remove `--insecure`, and use `--certificate-authorities=/path/to/ca.crt` as appropriate.

Fleet host: `https://192.168.64.1:8220`. Default Elasticsearch output: `http://192.168.64.1:9200`. The endpoint Agent sends log data directly to Elasticsearch and receives policy from Fleet. Set Custom Logs (Filestream) path to `/var/log/samba/log.samba` and dataset to `samba.auth`.

See [Fleet setup](../docs/06-fleet-and-elastic-agent.md) and [ingestion validation](../docs/07-log-ingestion.md).
