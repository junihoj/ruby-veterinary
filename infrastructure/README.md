# infrastructure

Development-only infrastructure for the ruby-veterinary workspace.

## Services

| Service | Image | Host ports | Purpose |
|---------|-------|-----------|---------|
| `postgres` | `postgres:16-alpine` | `5432` (not mapped) → container `5432`, published on host **`5433`** | Dev/test database (`ruby_veterinary_dev`, `ruby_veterinary_test`) |

> **MinIO deferred:** ADR-0009 selected MinIO for object storage, but MinIO Community
> Edition reached end of life (archived upstream February 2026) and its images were
> removed from Docker Hub in September 2026. The ADR is revisited before commerce
> milestone M3b; `docker-compose.dev.yml` ships postgres only until then.

The host port is **5433** so the compose database never collides with any locally
installed PostgreSQL service. API credentials live in `ruby-veterinary-api/.env`
(copied from `.env.example`).

## Commands

```bash
# from the repository root
docker compose -f infrastructure/docker-compose.dev.yml up -d      # start
docker compose -f infrastructure/docker-compose.dev.yml ps         # health (healthy = ready)
docker compose -f infrastructure/docker-compose.dev.yml down       # stop (keeps data)
docker compose -f infrastructure/docker-compose.dev.yml down -v    # stop and wipe volumes (fresh migrate check)
```

Connection strings:

```
postgresql://ruby:ruby_dev@localhost:5433/ruby_veterinary_dev
postgresql://ruby:ruby_dev@localhost:5433/ruby_veterinary_test
```
