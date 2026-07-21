# SEAD Explorer

**Read-only operational dashboard for SEAD nodes.** Provides human-comprehensible
visibility into a SEAD node's local perspective. It is not a consensus participant,
authority, registry, or protocol component.

## Architecture

```mermaid
graph LR
    subgraph "sead-service (Docker network)"
        SC[sead-core:8080]
        ES[edge-service:8081]
        SG[storage-gateway:8082]
        VR[verifier:8084]
    end

    subgraph "sead-explorer (Docker compose)"
        DB[(PostgreSQL 16)]
        API[FastAPI API]
        UI[React UI / nginx]
    end

    API -->|polling| SC
    API -->|optional| ES
    API -->|optional| SG
    API -->|optional| VR
    API -->|asyncpg| DB
    UI -->|HTTP /api| API
```

## Deploy

### Prerequisites

- Docker + Docker Compose plugin
- A running SEAD stack (see [stardome-sead](https://github.com/Stardome-technology/stardome-sead))

### Production deploy

```bash
# Ensure sead-network exists (only needed if running standalone without the
# SEAD stack; the SEAD stack auto-creates this network at startup)
docker network create sead-network 2>/dev/null || true

# Create .env with your configuration
# See reference below for all variables
docker compose -f docker-compose.remote.yml pull
docker compose -f docker-compose.remote.yml up -d

# Verify — use 127.0.0.1, not localhost (IPv6 resolution issues)
curl http://127.0.0.1:8086/health
curl http://127.0.0.1:3000/
```

### Configuration

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `SEAD_CORE_URL` | Yes | — | sead-core HTTP endpoint |
| `DATABASE_URL` | Yes | — | PostgreSQL connection string |
| `OBSERVER_ORG_ID` | Yes | — | Observer organization identity |
| `OBSERVER_NODE_ID` | Yes | — | Observer node identity |
| `EDGE_SERVICE_URL` | No | — | edge-service HTTP endpoint |
| `STORAGE_GATEWAY_URL` | No | — | storage-gateway HTTP endpoint |
| `VERIFIER_URL` | No | — | verifier-service HTTP endpoint |
| `IPFS_API_URL` | No | `https://ipfs.stardome.cloud` | IPFS node API endpoint |
| `INGESTION_INTERVAL_SECONDS` | No | 5 | Polling interval |
| `LOG_LEVEL` | No | INFO | Logging level |

#### Note about the SEAD Service URLs
If sead-eplorer is running in a docker-compose environment, use the following URLs:
```txt
SEAD_CORE_URL=http://sead-core:8080
EDGE_SERVICE_URL=http://edge-service:8081
STORAGE_GATEWAY_URL=http://storage-gateway:8082
VERIFIER_URL=http://verifier:8084
```
If sead-explorer is running outside of docker-compose, on the host machine, use the following URLs:
```txt
SEAD_CORE_URL=http://localhost:8080
EDGE_SERVICE_URL=http://localhost:8081
STORAGE_GATEWAY_URL=http://localhost:8082
VERIFIER_URL=http://localhost:8084
```
Else if sead-explorer is running outside of docker-compose, on a remote machine, use the remote IP:
```txt
SEAD_CORE_URL=http://<IP>:8080
EDGE_SERVICE_URL=http://<IP>:8081
STORAGE_GATEWAY_URL=http://<IP>:8082
VERIFIER_URL=http://<IP>:8084
```

## API Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/health` | Health check |
| GET | `/api/v1/peers` | List known peers |
| GET | `/api/v1/organizations` | List observed organizations |
| GET | `/api/v1/frontier` | Frontier summary counts |
| GET | `/api/v1/events` | Event list (paginated) |
| GET | `/api/v1/events/{event_id}` | Event detail |
| GET | `/api/v1/ipfs/status` | IPFS status |
| GET | `/metrics` | Prometheus metrics |
