# 05 — Elastic Security deployment

The original lab ran Elasticsearch 9.2.0 and Kibana 9.2.0 through Docker on the Mac. This example rebuild uses authentication with Elasticsearch HTTP and loopback Kibana HTTP. It intentionally reproduces lab transport choices; read [SECURITY.md](../SECURITY.md).

## Local setup

From the repository root:

```bash
cp .env.example .env
chmod 600 .env
```

Edit `.env` privately. Replace both passwords with fresh, distinct values and all three Kibana keys with independent random values of at least 32 characters. Keep those keys stable across restarts. Do not paste secrets into documentation or publish command output.

The Fleet placeholders can remain until [the next guide](06-fleet-and-elastic-agent.md). Confirm `192.168.64.1` exists on the host. Ensure Docker Desktop has memory beyond the Elasticsearch heap allocation.

```bash
docker compose --env-file .env -f configs/docker-compose.example.yml config --quiet
docker compose --env-file .env -f configs/docker-compose.example.yml up -d elasticsearch
docker compose --env-file .env -f configs/docker-compose.example.yml logs --tail=50 elasticsearch
curl --fail -u elastic http://192.168.64.1:9200/
```

`curl -u elastic` prompts for the password. Expect version `9.2.0` in the authenticated response. Plain HTTP exposes this authentication on the lab network. Keep the network isolated.

## Set the kibana_system password

The Compose password variable does not initialize the Elasticsearch built-in `kibana_system` user. Once Elasticsearch is ready, run the interactive reset tool and enter the exact local value from `.env`:

```bash
docker compose --env-file .env -f configs/docker-compose.example.yml exec elasticsearch \
  bin/elasticsearch-reset-password -u kibana_system -i
docker compose --env-file .env -f configs/docker-compose.example.yml up -d kibana
docker compose --env-file .env -f configs/docker-compose.example.yml logs --tail=50 kibana
```

Open `http://localhost:5601` and log in as `elastic` for initial lab setup. `kibana_system` is the server account, not a human Kibana login. Create narrower operator roles for regular use.

The example explicitly disables Elasticsearch HTTP TLS and enrollment auto-configuration so its configured HTTP URL and built-in user authentication agree. It is a single-node baseline, not a production cluster.

## Persistence and lifecycle

The `esdata` volume stores Elasticsearch data; `fleetstate` stores Fleet Server agent state when enabled. Keep .env and encryption keys safe and outside Git.

```bash
docker compose --env-file .env -f configs/docker-compose.example.yml stop
docker compose --env-file .env -f configs/docker-compose.example.yml start
```

Do not use `down -v` casually: it deletes the named lab volumes. Existing volumes may contain passwords from a previous run; changing `ELASTIC_PASSWORD` in .env does not reset an initialized cluster's password.

References: [Elasticsearch in Docker](https://www.elastic.co/docs/deploy-manage/deploy/self-managed/install-elasticsearch-docker-basic), [Kibana Docker configuration](https://www.elastic.co/docs/deploy-manage/deploy/self-managed/install-kibana-with-docker).

Next: [06 — Fleet and Elastic Agent](06-fleet-and-elastic-agent.md).
