# SoftwPortal — Technical Research & Build Guide

> **Purpose:** A deep, implementation-oriented research document that explains *how*
> SoftwPortal should be built, *what* to build at each layer, and *why* specific
> technologies and patterns are chosen. This complements `README.md` (PRD) and
> `PLANNING.md` (execution plan). Read this before writing code.
>
> **Scope:** Multi-tenant SaaS, verified companies/freelancers/clients, isolated
> workspaces, KYC, projects, documents, real-time chat, admin verification.
> **Stack under research:** Next.js + NestJS + PostgreSQL + Prisma + Socket.IO.
> **Compiled:** 2026-07-19.

---

## 1. Executive Summary of the Build Approach

SoftwPortal is not a single app — it is three concerns fused into one product:

1. **A verified identity layer** (KYC/KYB + admin approval) that gates access.
2. **A multi-tenant data plane** where every row belongs to exactly one workspace
   and can never leak to another.
3. **A real-time collaboration surface** (chat, presence, notifications) that must
   scale horizontally.

The recommended path is a **pnpm/Turborepo monorepo** with `apps/web` (Next.js),
`apps/api` (NestJS), and shared `packages/` (types, db, config). The API owns all
database access; the web talks to it only through a typed contract. This keeps the
security boundary in one place and makes the post-hackathon scale-up mechanical
rather than a rewrite.

For the hackathon, collapse the API into Next.js route handlers to ship fast. For
the startup track, extract `apps/api` into a standalone containerized NestJS service
behind a real WAF. The code does not change meaningfully — only the deployment shape.

---

## 2. Multi-Tenancy: The Highest-Stakes Decision

Tenant isolation is the single feature that, if wrong, ends the company. Three
patterns exist:

- **Shared schema + `tenant_id` column** (recommended default). All tenants share
  tables; isolation is enforced by a `workspaceId`/`tenant_id` column on nearly every
  row. Carries you to ~5,000 tenants on one instance, migrations run once, and
  cross-tenant analytics is a single `GROUP BY`. This is the PlanetScale/Cadence/
  peal.dev recommended starting point.
- **Schema-per-tenant**. Each tenant gets a Postgres schema, routed via `search_path`.
  Strong logical isolation and trivial GDPR erasure (`DROP SCHEMA`), but ops cost
  explodes past ~500–1,000 schemas due to `pg_catalog` contention.
- **Database-per-tenant** (silo). Maximum isolation and per-tenant point-in-time
  recovery, but highest cost and hard connection pooling.

**Chosen approach:** shared schema with `workspaceId` for the long tail, with a clear
migration path to dedicated schema/DB for enterprise/paying tiers later (the "hybrid"
model mature SaaS lands on). **Row-Level Security (RLS)** is used as *defense-in-depth*
only — not the sole control. Pattern: enable RLS, create a policy keyed on
`current_setting('app.workspace_id')`, and `SET LOCAL app.workspace_id = ...` inside
every transaction (never a bare `SET` in a pooled connection). Use `FORCE ROW LEVEL
SECURITY` in tests so superuser bypass cannot mask a bug. Composite unique indexes
such as `(workspaceId, email)` prevent existence leaks.

**Critical pitfalls to engineer against:** forgetting the `WHERE workspaceId` filter;
connection-pool context leakage between requests; UUID enumeration (always use
`gen_random_uuid()`); noisy-neighbor queries (set `statement_timeout` and
`idle_in_transaction_session_timeout`).

---

## 3. Authentication & Sessions

Security starts at login. Use **Argon2id** (OWASP/NIST first choice; memory-hard,
16-byte salt, memoryCost ~64 MiB, timeCost 3, parallelism 4). Reject breached
passwords via the HaveIBeenPwned k-anonymity range API. NIST SP 800-63-4 (Aug 2025)
governs current guidance.

**Token model:** short-lived JWT access (15 min) signed with **RS256 asymmetric
keys** (private signs, public/JWKS verifies, rotate ~90 days) + an opaque 256-bit
refresh token stored **SHA-256 hashed** in a `Session` table, delivered in an
`HttpOnly` + `Secure` + `SameSite=Strict` cookie (never localStorage). On every
refresh, **rotate** the refresh token and track a token *family* UUID; if a
previously-rotated token reappears, revoke the whole family (theft signal) and force
re-login. Revoke on password reset and logout. Early access-token revocation uses a
Redis `jti` blacklist with TTL equal to token lifetime.

**MFA:** offer **TOTP (RFC 6238)** as baseline and **WebAuthn/FIDO2 passkeys** as the
phishing-resistant default for Super Admin and Company owners (NIST AAL2+). Treat SMS
as fallback only (SSIM/SIM-swap risk). Encrypt TOTP secrets with AES-256-GCM, allow a
±1 step window, rate-limit attempts, and store single-use recovery codes hashed.

**Session hygiene:** rotate session ID on privilege change; idle 15–30 min, absolute
8–24 h; server-side sessions in Redis provide a real kill switch. Use generic login
errors (no user enumeration) and per-account + per-IP rate limiting with backoff.

---

## 4. KYC & Identity Verification

SoftwPortal verifies *companies* (KYB — business + beneficial owner) and *individuals*
(KYC — freelancers, client owners). Start with a vendor rather than building document
review in-house:

- **Persona** — best developer experience, modular flows, ~$1–2/verification (used by
  Brex/OpenAI). Good default.
- **Stripe Identity** — $1.50/verification, fastest if already on Stripe, US-focused,
  no AML/KYB.
- **Veriff** — strong doc-capture UX, ~6s median decision.
- **Sumsub** — all-in-one KYC+KYB+AML+transaction monitoring; strong for broader
  compliance.
- **Onfido (Entrust)/Jumio** — enterprise, sales-led, deepest AML/PEP coverage.

**Build pattern:** the platform stores only KYC *metadata and status*; the vendor
holds raw documents. For the hackathon, stub the vendor with an admin "approve" button
and store uploaded docs locally. Post-hackathon, integrate Persona/Stripe and encrypt
any documents you do store at rest (SSE-KMS).

**Compliance reality:** GDPR is a property of the entire data flow — where data lives,
retention, who can decrypt, region. Minimize retained PII, set configurable retention,
and expect SOC 2 Type II + ISO 27001 + GDPR. Use a **quarantine → scan → promote**
pipeline for uploaded identity documents.

---

## 5. Real-Time Communication at Scale

Socket.IO is correct for chat/presence/notifications. The naive single-server setup
breaks the moment you run two instances, so design for the adapter from day one.

- **Adapter:** `@socket.io/redis-adapter`, preferably the **sharded adapter**
  (`createShardedAdapter()`, Redis 7+ sharded pub/sub) — it partitions channels by
  hash slot and scales linearly past 60–80k connections where classic pub/sub becomes
  the bottleneck. The `redis-streams-adapter` additionally survives Redis disconnects
  without message loss.
- **Transport:** set `transports: ["websocket"]` to drop long-polling (it triples CPU
  at scale and removes the need for sticky sessions when pure WS is used). Use
  `ioredis`, not the `redis` package (subscription-restore bug).
- **Rooms:** scope broadcasts with `io.to(workspaceId).emit()`; rooms are the primary
  broadcast primitive. Authenticate the socket with the same JWT.
- **Presence:** store socket IDs per user in Redis with a 30s heartbeat TTL; `hlen = 0`
  → offline. A sorted set with `ZADD` timestamps answers "online in last 60s".
- **Reconnection:** the client auto-reconnects with exponential backoff; v4.6+
  connection-state recovery restores room membership and pending messages. Rate-limit
  connections per `user_id` in Redis middleware.
- **Capacity:** ~30k connections/node default, ~50k with `uws`; RAM-bound (~8 KB/conn).
  Cross-region means a separate Redis cluster + NATS/Kafka federation — never stretch
  one Redis across regions.

---

## 6. File Storage Security

Documents are a classic RCE and data-leak vector if mishandled.

- **Upload flow:** browser requests a **presigned URL** from the API, uploads directly
  to S3/R2 (no bytes through your server), then calls a `finalize` endpoint. Keep
  buckets **private**; serve downloads only via short-lived signed URLs (upload 30–120s
  TTL, download 60–300s). Bind the exact key, restrict `ContentType`, add
  `content-length-range`, and least-privilege the signing IAM role to a specific
  prefix. Redact signed query params from all logs — they are bearer tokens.
- **Encryption:** SSE-KMS with a customer-managed key; tighten the KMS key policy to
  `kms:ViaService` S3 only.
- **Scanning:** **quarantine → scan → promote**. Use AWS GuardDuty Malware Protection
  for S3 (EventBridge → Lambda promotes clean files), ClamAV in a worker, or managed
  AV. Keep scanning off the request path (async worker).
- **Validation:** never trust client `Content-Type` or filename. Allowlist extensions
  → expected MIME → **magic-byte** verification (JPEG `FFD8FF`, PNG `89504E47`, PDF
  `%PDF-`). Generate UUID object keys (no raw filenames → path traversal). Re-encode
  images to strip EXIF/GPS. Treat SVG/HTML/JS as active code — block or sanitize.
- **Serving:** `Content-Disposition: attachment`, `X-Content-Type-Options: nosniff`.
  Per-user quotas + rate limits on upload endpoints; log sha256/type/size/scanner
  result.

---

## 7. Authorization: RBAC, IDOR & Tenant Context

**RBAC:** **CASL** is the idiomatic NestJS choice (`@casl/ability`,
`createMongoAbility`). Declare `can`/`cannot` rules with conditions
(`{ workspaceId: user.workspaceId }`), field-level rules, and `manage`/`all`
wildcards. One `CaslAbilityFactory` is the source of truth; a `@CheckPolicies`
decorator + `PoliciesGuard` runs *after* the auth guard. Do class-level checks in the
guard, **instance-level ownership checks in the service** where the entity is loaded.

**Tenant-aware authorization:** plain CASL forgets `workspaceId` silently. Use a
tenant-aware layer (`nest-warden`, `@trishchuk/nest-casl`, or `@nestarc/rbac`) or
resolve the workspace from the JWT into a CLS (continuation-local-storage) context that
auto-filters every query. Two-guard order: auth guard sets `request.user` → policies/
tenant guard enforces scope.

**IDOR / BOLA (the #1 API risk three editions running):** use UUIDs (never sequential
IDs), enforce ownership/`workspaceId` checks on *every* object-access endpoint, and
validate the requester's scope against the resource's `workspaceId` server-side. This
is where most SaaS breaches actually happen.

---

## 8. API Security & OWASP Top 10 (2025)

The OWASP API Security Top 10 (2025) added **SSRF (API7)** and **Unsafe Consumption of
APIs (API10)**; BOLA (API1) remains #1. Key defenses:

- **BOLA/IDOR:** UUIDs + server-side ownership/tenant checks (§7).
- **BOPLA (mass assignment / over-exposure):** strict DTO field allow-lists
  (`stripUnknown`); never return full entities; never accept unmapped fields.
- **Rate limiting:** per-IP **and** per-account via Redis (`@nestjs/throttler`,
  express-rate-limit + RedisStore). Stricter on auth (5 attempts/15 min + backoff).
  Return `429` + `Retry-After` + `X-RateLimit-*`.
- **Validation/sanitization:** Zod/Joi at gateway + app + DB; parameterized queries /
  ORM only (no string interpolation); sanitize HTML (Bleach/DOMPurify).
- **SSRF:** allow-list outbound domains; block RFC1918/loopback/cloud-metadata
  (169.254.169.254) IPs at the WAF; never process responses from user-supplied URLs.
- **WAF + headers:** AWS WAF or Cloudflare (OWASP Core Rule Set, Bot Fight Mode).
  Helmet with strict CSP (`default-src 'self'`), HSTS preload,
  `X-Content-Type-Options: nosniff`, `Referrer-Policy`, `Permissions-Policy`. CORS:
  explicit origin allow-list, no wildcard with credentials.
- **Inventory:** discover shadow/zombie APIs and assign owners; schema-validate all
  third-party responses (Unsafe Consumption). Cequence's 2025 insight: API risks are
  *vulnerabilities in your code*, not attacks to block — fix in code + CI/CD, use WAF
  only as a buy-time bridge.

---

## 9. Architecture, Deployment & CI/CD

- **Monorepo:** pnpm/Bun workspaces + **Turborepo** (or Nx). Layout:
  `apps/web` (Next.js), `apps/api` (NestJS), `packages/{types,db,config,ui}`. Keep DB
  access only in the API. Generate a typed contract (Swagger → Orval for React Query,
  or tRPC, or shared `@repo/types`) so the frontend and backend never drift. Root
  Biome/ESLint + Husky pre-commit. `next-forge` is Vercel's production Turborepo
  reference.
- **NestJS hardening:** Helmet, `@nestjs/throttler`, global `AllExceptionsFilter`,
  `ClsModule` (correlation IDs), `@nestjs/terminus` health endpoints, Zod validation
  pipe, CQRS for complex domains. JWT RS256; separate key pairs per app/role
  (`platform` claim).
- **Deployment:** Web → **Vercel** (preview deploys, edge auth middleware). API →
  **Render / Fly.io / Railway / ECS Fargate** (containerized, WebSocket-friendly —
  Vercel is poor for Socket.IO long connections). DB → **Neon** (serverless Postgres,
  branching per preview). Redis → Upstash/Elasticache.
- **CI/CD:** GitHub Actions — lint + test (Postgres + Redis services for E2E) on PR;
  on merge to main: build → push image → **run the migration task BEFORE the rolling
  deploy** (zero-downtime; new code must never hit an un-migrated schema) → deploy.
  Remote caching (Turborepo), conventional commits + commitlint.
- **Secrets:** never plaintext env. **AWS Secrets Manager / Doppler / Vercel+Rnder
  encrypted env**. CI uses **OIDC** (no long-lived keys). Rotate JWT keys via JWKS;
  KMS for field encryption; least-privilege IAM. Scan deps (Snyk/Dependabot) in CI.

---

## 10. 2025–2026 Trends to Plan Around

- Sharded Redis adapter for Socket.IO (linear scaling past 60–80k conns).
- RLS as default-but-secondary isolation, layered on `tenant_id`.
- Argon2id + passkeys/WebAuthn as phishing-resistant MFA.
- Reusable credentials & decentralized KYC data architectures (lower GDPR liability).
- OWASP API Top 10 2025 adds SSRF + Unsafe Consumption of APIs.
- Hybrid tenancy (shared schema for the long tail, dedicated tiers for enterprises).
- Contract-driven typing in Turborepo monorepos.
- CI OIDC + pre-deploy migration tasks as standard.

---

## 11. Build Sequencing (Practical)

1. **Foundation:** monorepo, Postgres + Prisma, `workspaceId` on every table, auth
   (Argon2id + RS256 JWT + refresh rotation).
2. **Isolation proof:** TenantContext middleware + CASL + a test suite that asserts
   cross-tenant queries fail.
3. **Verification:** KYC submission + admin approval (stub vendor for hackathon).
4. **Collaboration:** projects, documents (presigned upload + scan + signed URL
   download), Socket.IO chat with Redis adapter.
5. **Hardening:** WAF, CSP/Helmet, rate limiting, MFA, audit logging, secret scanning.
6. **Scale:** extract API to container, sharded Redis, Neon branching, CI/CD with
   pre-deploy migrations.

This research gives a concrete, defensible blueprint: shared-schema multi-tenancy with
RLS defense-in-depth, Argon2id + RS256 + rotating refresh tokens with MFA, vendor-based
KYC with encrypted quarantine storage, Socket.IO on a sharded Redis adapter, presigned
S3/R2 with malware scanning, CASL RBAC with tenant-aware IDOR protection, and a WAF +
CSP + rate-limit + CI/OIDC security posture — deployed as a Turborepo monorepo on
Vercel + Render/Fly + Neon.

---

## Navigation

- **Previous:** [PLANNING.md](./PLANNING.md) (the two-track plan this research supports).
- **Next read:** [Architecture.md](./Architecture.md) — how the system is structured.
- **Then:** [Database Design.md](./Database%20Design.md) → [API Documentation.md](./API%20Documentation.md) → [UI-UX.md](./UI-UX.md).
- **Security detail:** [Security.md](./Security.md).
- **Master map:** [INDEX.md](./INDEX.md).
