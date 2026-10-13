# infrastructure

Development-only infrastructure for the ruby-veterinary workspace.

## Services

| Service | Image | Host ports | Purpose |
|---------|-------|-----------|---------|
| `postgres` | `postgres:16-alpine` | container `5432`, published on host **`5433`** | Dev/test database (`ruby_veterinary_dev`, `ruby_veterinary_test`) |
| `seaweedfs` | `chrislusf/seaweedfs:3.99` (pinned by digest) | container `8333` (S3 API) and `9333` (master UI), published as-is | Dev object storage behind the S3 API (ADR-0016) |

The host port is **5433** so the compose database never collides with any locally
installed PostgreSQL service. API credentials live in `ruby-veterinary-api/.env`
(copied from `.env.example`).

> **Why SeaweedFS, not MinIO:** ADR-0009 originally selected MinIO, but MinIO
> Community Edition reached end of life (archived upstream February 2026), its
> Docker Hub images were deleted entirely in September 2026 (the repository
> itself 404s), and `dl.min.io` returns HTTP 410. ADR-0016 supersedes ADR-0009
> and selects SeaweedFS, which keeps the same S3-compatible contract (signed
> direct uploads/downloads, private buckets, no file bytes through the API).
> The image is pinned by tag **and digest** so a vanished upstream tag cannot
> break a fresh clone.

SeaweedFS runs single-process (`weed server -s3`) with anonymous S3 access on
localhost — development only; the deployment milestone configures access keys
and keeps the S3 port off the public network (`deployment-architecture.md`).

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
S3 API:           http://localhost:8333  (ListBuckets XML = ready)
SeaweedFS master: http://localhost:9333  (UI + /cluster/status)
```
