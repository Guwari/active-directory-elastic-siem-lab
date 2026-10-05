# 06 — Fleet and Elastic Agent

## Fleet settings

In Kibana → Management → Fleet → Settings:
- Set the Fleet Server host to `https://192.168.64.1:8220`.
- Set the default Elasticsearch output to `http://192.168.64.1:9200`.

Use the bridge address for endpoint policies. `localhost` on DC01 means DC01; the Docker name `elasticsearch` is not resolvable by the Debian guest. Kibana and Fleet Server containers can use `http://elasticsearch:9200` internally.

## Fleet Server

In Fleet, choose Add Fleet Server and create a policy with the Fleet Server integration. For the sample Compose profile, privately save its policy ID and newly generated service token in `.env` as `FLEET_SERVER_POLICY_ID` and `FLEET_SERVER_SERVICE_TOKEN`.

The example uses a containerized Fleet Server as a reproducible option. The retained lab recap confirms the HTTPS endpoint but does not establish the exact original Fleet Server startup command.

```bash
docker compose --env-file .env -f configs/docker-compose.example.yml --profile fleet up -d fleet-server
docker compose --env-file .env -f configs/docker-compose.example.yml logs --tail=50 fleet-server
```

Without explicit certificate/key configuration, Fleet Server generates a self-signed certificate. This rebuild therefore uses the lab Agent `--insecure` flag. Do not set Fleet Server's own insecure HTTP mode: the recorded endpoint is HTTPS. Use a trusted CA and SAN-correct certificate for a hardened rebuild.

Validate Fleet Server in Fleet. Reachability alone does not prove a working Fleet policy.

## DC01 policy and enrollment

1. Create an Agent policy named **DC01 Linux Policy**.
2. Add **Custom Logs (Filestream)** to that policy using [the ingestion guide](07-log-ingestion.md).
3. Choose Add agent for that specific policy; generate a fresh enrollment token.
4. Follow [the sanitized Agent installation example](../configs/elastic-agent.example.md) on DC01.
5. Wait for the policy update and confirm **Healthy** in Fleet.

Fleet service tokens authenticate Fleet Server; enrollment tokens enroll endpoint Agents. They are not interchangeable. After enrollment the Agent uses issued credentials for check-ins and outputs.

## Connectivity checks from DC01

```bash
curl --connect-timeout 5 -I http://192.168.64.1:9200
openssl s_client -connect 192.168.64.1:8220 </dev/null
sudo elastic-agent status
```

An unauthenticated Elasticsearch 401 response confirms the service responds, not successful ingestion. A self-signed certificate verification error in the TLS check is expected for the recorded lab and is not a reason to disable broader security controls.

Inspect Fleet status and Agent logs privately. Do not publish full diagnostics: they can contain credentials and sensitive configuration.

Reference: [Elastic Agent container deployment](https://www.elastic.co/docs/reference/fleet/elastic-agent-container), [Elasticsearch output](https://www.elastic.co/docs/reference/fleet/elasticsearch-output).

Next: [07 — Log ingestion](07-log-ingestion.md).
