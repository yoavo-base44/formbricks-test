# AGENTS.md — Formbricks

## Project overview
Formbricks is a monorepo (pnpm workspaces + Turborepo) built with Next.js 16 (App Router, Turbopack), Prisma 7, PostgreSQL (pgvector), Redis/Valkey, and a Go-based Hub API.

## Repository layout
- `apps/web` — Next.js 16 app (main UI, port 3000)
- `apps/storybook` — Storybook instance
- `packages/database` — Prisma schema, migrations, seed
- `packages/*` — shared libraries (types, logger, cache, storage, email, jobs, ai, surveys, js-core, etc.)
- `docker/` — Cube config, rustfs-init script, production compose
- `docker-compose.dev.yml` — official dev infra (Postgres, Valkey, MailHog, Hub, Cube, RustFS)
- `docker-compose.base44.yml` — Base44 full-stack dev environment (all-in-one)

## Local dev setup (Base44)
```bash
docker compose -f docker-compose.base44.yml up -d
```
This brings up:
- PostgreSQL (pgvector:pg18) on port 5432
- Valkey (Redis-compatible) on port 6379
- MailHog on ports 1025/8025
- Formbricks Hub API on port 8080
- Cube analytics on port 4000
- RustFS (S3-compatible storage) on port 9000
- Next.js dev server on port 3000 (Turbopack, live reload)

## Key quirks
- The `web` service installs deps, builds workspace packages, pushes the Prisma schema, then starts `next dev --turbopack`.
- First start takes ~2 minutes (pnpm install + builds). Subsequent restarts are fast thanks to the `node_modules_vol` volume.
- Prisma migrations use a custom runner (`packages/database/src/scripts/apply-migrations.js`); for dev, `prisma db push` is sufficient.
- `allowedDevOrigins` in `next.config.mjs` includes the `BASE44_PUBLIC_HOST_SUFFIX` for the preview proxy.
- Hub (Go service) handles feedback/response analysis; runs as a prebuilt image.
- Cube provides analytics/metrics via a semantic layer.
- No external secrets are required to boot — all services run locally with generated dev credentials.

## Testing
```bash
# Unit tests
docker compose -f docker-compose.base44.yml exec web pnpm test

# Lint
docker compose -f docker-compose.base44.yml exec web pnpm lint
```

## Common operations
- Schema changes: edit `packages/database/schema.prisma`, then run `prisma db push` inside the web container.
- Add a new workspace package: create under `packages/`, add to `pnpm-workspace.yaml`, rebuild.
