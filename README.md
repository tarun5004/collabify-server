# Collabify Server — project fork

**A TypeScript collaboration backend using Bun, Hono, GraphQL, PostgreSQL, Redis and WebSockets.**

This repository is a fork of [team7-collabify/collabify-server](https://github.com/team7-collabify/collabify-server), originating from [test-organization-sheryians/collabify-server](https://github.com/test-organization-sheryians/collabify-server). Credit for the upstream implementation belongs to its contributors. This fork's README documents its architecture and development entry points; it does not claim original authorship of the upstream application.

## Backend overview

The codebase brings together workspaces, projects, issue tracking, chat, collaborative pages, whiteboards, notifications and file storage.

```text
HTTP / GraphQL / WebSocket clients
               |
          Hono + GraphQL Yoga
               |
      Feature modules and authorization
               |
    PostgreSQL / Prisma + Redis / BullMQ
```

| Area | Source |
| --- | --- |
| HTTP startup, health and WebSocket gateway | [`src/app/server.ts`](src/app/server.ts) |
| Feature modules | [`src/modules/`](src/modules/) |
| Database schema and migrations | [`prisma/`](prisma/) |
| Database, Redis and WebSocket infrastructure | [`src/infra/`](src/infra/) |
| Required configuration | [`src/shared/config/env.ts`](src/shared/config/env.ts) |

## Local development

Prerequisites: **Bun 1.3.9**, Docker Compose, PostgreSQL 16, Redis 7 and a Clerk development application. Optional storage and email integrations require their own configuration.

```bash
git clone https://github.com/tarun5004/collabify-server.git
cd collabify-server
bun install
docker compose up -d postgres redis
```

Create an untracked `.env` in the repository root. The environment schema is authoritative; required variables are:

- `DATABASE_URL` — local Compose PostgreSQL listens on host port `5434`.
- `REDIS_URL` — local Compose Redis listens on host port `6379`.
- `CLERK_SECRET_KEY`, `CLERK_PUBLISHABLE_KEY`, `CLERK_WEBHOOK_SIGNING_SECRET` — development values from your Clerk application.
- `EMAIL_FROM` — a valid development sender address; email and push providers default to `console`.

`PORT` defaults to `3001`; `FRONTEND_URL` defaults to `http://localhost:3000`. Consult the schema for optional provider settings. Do not use production credentials or point migrations at a shared database.

With the local services healthy and environment configured:

```bash
bunx prisma generate
bunx prisma migrate deploy
bun run gen
bun run --hot src/app/server.ts
```

The direct server command uses the PostgreSQL/Redis services started above. `bun run dev` also starts the full Compose stack, including the separately configured Nginx service.

## Entry points and checks

- `GET /` — server response.
- `GET /health` — PostgreSQL and Redis readiness.
- `/graphql` — GraphQL endpoint.
- `/ws` — WebSocket gateway.

```bash
bun run typecheck
bun run lint
bun test ./src
bun run build
```

These are available project commands, not a claim that this fork has passed a fresh runtime or deployment verification. Provider-dependent features require their corresponding services. Upstream deployment workflows target upstream infrastructure; they are not local setup steps.
