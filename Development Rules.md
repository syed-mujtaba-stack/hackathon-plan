# SoftwPortal — Development Rules

> Non-negotiable engineering conventions for everyone contributing to SoftwPortal.
> Enforced in code review and CI where possible.

---

## 1. General

- **Language:** TypeScript everywhere. No `any` in shared/app code (use `unknown` +
  narrowing). Strict mode on.
- **Single source of truth:** `packages/types` for API contracts; `packages/db` for
  schema. Never duplicate shapes.
- **Commits:** Conventional Commits (`feat:`, `fix:`, `security:`, `docs:`,
  `chore:`). Scanned by commitlint.
- **Branches:** `main` (protected) ← `feature/<ticket>` ← PR. No direct pushes to
  `main`.
- **PRs:** require 1 approval, green CI, no secret-scan failures. Link the issue.

---

## 2. Backend (NestJS)

- **Validation:** every controller input goes through a zod DTO pipe; `stripUnknown`.
- **Authz:** every route has auth guard + RBAC guard; tenant guard sets CLS.
- **Tenant safety:** never build a query without `workspaceId` from context. If you
  write raw SQL, you must set `SET LOCAL app.workspace_id` and add a test.
- **No secrets:** never log tokens, passwords, file keys with PII. Use redaction.
- **Errors:** throw domain exceptions; let `AllExceptionsFilter` format. No stack to
  client.
- **Testing:** unit tests for services; integration tests for tenant isolation + IDOR.

---

## 3. Frontend (Next.js)

- **Types:** generate React Query hooks from OpenAPI (Orval); do not hand-type API.
- **Forms:** react-hook-form + zod; show inline errors; never trust client validation.
- **Secrets:** `NEXT_PUBLIC_*` only for non-sensitive config (API URL). Never tokens.
- **Accessibility:** semantic HTML; keyboard support; AA contrast.
- **State:** server state via React Query; minimal local state; no global auth store
  holding tokens in localStorage.

---

## 4. Database

- **Migrations:** only via Prisma migrate; reviewed; run before deploy.
- **Tenant columns:** every tenant-scoped table has `workspaceId` FK + index.
- **RLS:** keep policies in sync with schema changes; `FORCE` in tests.
- **No destructive ops** without review (no `DROP`/`DELETE` without WHERE in prod).

---

## 5. Security Musts

- Argon2id for passwords; RS256 JWT; rotating refresh; MFA for admins.
- All user input validated + sanitized; parameterized queries only.
- Files: presigned upload, signed download, MIME allow-list, scan before promote.
- Rate limits on auth/invite/upload; CSP + WAF in front.
- Dependency + secret scanning in CI; patch critical CVEs within SLA.

---

## 6. Real-Time

- Authenticate socket with JWT; reject on failure.
- Join only rooms the user's `workspaceId` permits.
- No sensitive data in socket payloads beyond what REST would return.

---

## 7. CI/CD

- Lint + typecheck + test (Postgres + Redis services) on every PR.
- Build image; **run migration task before rolling deploy**.
- OIDC for cloud auth; no long-lived keys in CI.
- Preview deploy per PR (Neon branch).

---

## 8. Docs & Communication

- Keep `README.md`, `PLANNING.md`, `RESEARCH.md`, and architecture docs in sync.
- Update `API Documentation.md` when endpoints change.
- Ticket-first: no code without a linked task in `Module-wise Tasks.md` / sprint board.

---

## 9. Definition of Done

- [ ] Code reviewed and approved
- [ ] Tests pass (incl. tenant isolation + security checks)
- [ ] Lint/typecheck clean
- [ ] Docs updated
- [ ] No new secrets; CI green
- [ ] Demoed (for hackathon) or verified in staging

---

## Navigation

- **Previous:** [Security.md](./Security.md) (what we must enforce).
- **Plan the work:** [Roadmap.md](./Roadmap.md) → [Sprint Planning.md](./Sprint%20Planning.md) → [Module-wise Tasks.md](./Module-wise%20Tasks.md).
- **Start here if new:** [INDEX.md](./INDEX.md).
