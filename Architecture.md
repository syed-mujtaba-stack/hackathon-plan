# SoftwPortal — Architecture

> Defines the system architecture: deployment topology, service boundaries, data flow,
> and cross-cutting concerns. Companion docs: `README.md` (PRD), `PLANNING.md`
> (execution), `RESEARCH.md` (build guide), `Database Design.md`, `Security.md`.

---

## 1. Architectural Principles

1. **Security boundary in one place** — all database access lives in the API. The web
   client never touches the DB or object storage directly.
2. **Typed contract between frontend and backend** — no implicit coupling; schema
   drift is caught in CI.
3. **Tenant context is implicit, never trusted from the client** — resolved from the
   JWT and enforced server-side on every query.
4. **Stateless API, stateful realtime** — REST/JSON for commands, Socket.IO for events.
5. **Defense in depth** — network, app, data, and tenant layers each carry controls.

---

## 2. System Topology

```
                         ┌────────────────────────────┐
   Browser (Next.js) ───▶│  Vercel Edge (auth MW, CSP) │
                         └────────────┬───────────────┘
                                      │ HTTPS / WSS
              ┌───────────────────────┼───────────────────────┐
              ▼                       ▼                       ▼
      ┌──────────────┐      ┌──────────────────┐     ┌──────────────────┐
      │  API (NestJS)│◀────▶│ Realtime (Socket) │     │  Admin / Webhooks│
      │  REST + auth │      │  Socket.IO + Redis│     │  KYC vendor cb   │
      └──────┬───────┘      └────────┬──────────┘     └────────┬─────────┘
             │                       │                        │
             ▼                       ▼                        ▼
      ┌──────────────┐      ┌──────────────────┐     ┌──────────────────┐
      │ PostgreSQL   │      │ Redis (adapter,  │     │ Object Storage   │
      │ (Neon, RLS)  │      │ presence, cache) │     │ (S3/R2, KMS)      │
      └──────────────┘      └──────────────────┘     └──────────────────┘
```

- **Web:** Next.js on Vercel (preview deploys, edge middleware for auth redirect).
- **API:** NestJS container on Render / Fly.io / Railway / ECS Fargate.
- **Realtime:** same process or sidecar running Socket.IO with a Redis adapter.
- **DB:** Neon (serverless Postgres) with RLS + branch-per-preview.
- **Cache/Realtime state:** Upstash / Elasticache Redis.
- **Storage:** S3 or Cloudflare R2, private buckets, signed URLs only.

---

## 3. Monorepo Layout

```
softwportal/
  apps/
    web/            # Next.js (React, TS, Tailwind, shadcn/ui)
    api/            # NestJS (REST + Socket.IO)
  packages/
    types/          # Shared API contract (DTOs, enums)
    db/             # Prisma schema + migrations
    config/         # Env schema (zod), constants
    ui/             # Shared React components (optional)
  infra/            # Dockerfile, CI workflows, IaC (optional)
```

Keep `packages/types` as the single source of truth for request/response shapes.
Generate React Query hooks from the OpenAPI spec (Orval) so the client and server
never drift.

---

## 4. Layers (API)

| Layer | Responsibility |
|-------|----------------|
| **Gateway / Controllers** | Auth guard (JWT), RBAC guard, tenant guard, validation, rate limit |
| **Services** | Business logic (KYC, Workspace, Project, Document, Notification, Activity) |
| **TenantContext** | Resolves `workspaceId` from JWT into CLS; forces it on queries |
| **Repositories / Prisma** | Parameterized data access; RLS `SET LOCAL` per transaction |
| **Realtime** | Socket auth, room join, presence, event emit |
| **Integrations** | KYC vendor, email, object storage, payments (later) |

---

## 5. Request Lifecycle

1. Client sends `Authorization: Bearer <access>` (or refresh cookie).
2. Auth guard validates JWT, loads `request.user` (id, role, workspaceId).
3. Tenant guard / CLS stores `workspaceId`; every query is scoped to it.
4. RBAC guard checks the ability for the route (CASL).
5. Controller validates DTO (zod); service executes; repository writes.
6. `ActivityLog` records the mutation; response returns only allowed fields.

---

## 6. Real-Time Design

- Socket authenticates with the same JWT on connect.
- Room = `workspace:{id}` and `project:{id}`.
- Adapter = `@socket.io/redis-adapter` (sharded for scale).
- Presence stored in Redis with heartbeat TTL.
- Events: `join`, `message:new`, `notification:new`, `presence:update`, `typing`.

---

## 7. Cross-Cutting Concerns

- **Observability:** correlation IDs (ClsModule), structured logs, health endpoints
  (`/health/live`, `/health/ready` via `@nestjs/terminus`).
- **Config:** zod-validated env; fail fast on missing required vars.
- **Errors:** global `AllExceptionsFilter` returns generic messages; never leak stack.
- **Migrations:** run before rolling deploy (zero-downtime).

---

## 8. Environments

| Env | Purpose | Notes |
|-----|---------|-------|
| `local` | Dev | Docker Postgres + Redis |
| `preview` | PR previews | Neon branch per PR |
| `staging` | Pre-prod | Mirrors prod, real KYC sandbox |
| `prod` | Live | WAF, KMS, monitoring |

---

## 9. Scaling Strategy

- Horizontal API replicas behind load balancer.
- Sharded Redis adapter past ~60–80k socket connections.
- Neon compute autoscaling; read replicas for analytics.
- Hash-partition large tables on `workspaceId`.
- CDN (CloudFront) in front of static assets and signed downloads.
