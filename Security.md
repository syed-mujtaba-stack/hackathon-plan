# SoftwPortal — Security

> Consolidated security specification. Implements the threat model from `PLANNING.md`
> §5 and `RESEARCH.md` §3, §6, §7, §8. This is the authoritative security control set.

---

## 1. Security Posture Goals

- No plaintext credentials; no cross-tenant data leakage; no unauthenticated mutation.
- Defense in depth across network, app, data, and tenant layers.
- Aligns with OWASP API Top 10 (2025) and NIST SP 800-63-4.

---

## 2. Transport & Network

- TLS 1.2+ everywhere; WSS for sockets; HSTS (preload).
- **WAF:** Cloudflare or AWS WAF (OWASP Core Rule Set, Bot Fight Mode, IP reputation).
- **Headers:** strict CSP (`default-src 'self'`), `X-Content-Type-Options: nosniff`,
  `X-Frame-Options: DENY`, `Referrer-Policy: no-referrer`, `Permissions-Policy`.
- **CORS:** explicit origin allow-list; no wildcard with credentials.
- **SSRF:** block RFC1918/loopback/169.254.169.254; allow-list outbound domains.

---

## 3. Authentication

- **Hashing:** Argon2id (memoryCost 64 MiB, timeCost 3, parallelism 4, 16-byte salt).
- **Breached passwords:** rejected via HIBP k-anonymity.
- **JWT:** RS256 access (15 min) + opaque refresh (7–30d) stored SHA-256 hashed.
- **Refresh cookie:** `HttpOnly` + `Secure` + `SameSite=Strict`.
- **Rotation:** token family UUID; reused rotated token → revoke family + force login.
- **Revocation:** on password reset, logout, compromise (Redis `jti` blacklist).
- **MFA:** TOTP (RFC 6238) + WebAuthn passkeys for admins/owners; SMS fallback only;
  TOTP secret AES-256-GCM; recovery codes hashed single-use.
- **Lockout:** per-account + per-IP rate limit, exponential backoff; temporary (DoS-safe).

---

## 4. Authorization & Multi-Tenancy

- **RBAC:** CASL abilities; `@CheckPolicies` + `PoliciesGuard` after auth guard.
- **TenantContext:** `workspaceId` resolved from JWT into CLS; forced on every query.
- **RLS:** enabled as defense-in-depth; `SET LOCAL app.workspace_id` per transaction;
  `FORCE ROW LEVEL SECURITY` in tests.
- **IDOR/BOLA:** UUIDs (never sequential); ownership + `workspaceId` checked on every
  object endpoint server-side.
- **DTO allow-list:** `stripUnknown`; never return full entities; never accept unmapped
  fields (BOPLA).

---

## 5. Input / Output

- **Validation:** zod at gateway + app + DB; length/size caps; char allow-lists.
- **Queries:** Prisma parameterized only; no string interpolation (SQLi-proof).
- **Sanitization:** DOMPurify/Bleach for chat/comments; output encoding.
- **Errors:** generic messages; no stack traces; no secrets in logs.

---

## 6. File Security

- **Presigned upload** (30–120s TTL) directly to S3/R2; private buckets.
- **Signed download** (60–300s), exact key, `ContentType` + `content-length-range`
  locked; least-privilege signing IAM.
- **Encryption:** SSE-KMS, customer-managed key, `kms:ViaService` S3 only.
- **Scanning:** quarantine → scan (GuardDuty/ClamAV) → promote; async worker.
- **Validation:** extension allow-list → MIME → magic-byte; UUID object keys; strip
  EXIF/GPS; treat SVG/HTML/JS as active code.
- **Serving:** `Content-Disposition: attachment`; per-user quotas + rate limits.

---

## 7. Invitations

- Token = `crypto.randomBytes(32)` hex; **single-use**; **TTL** (7d); bound to email;
  revocable; guessed/reused tokens invalid.

---

## 8. Rate Limiting & Monitoring

- Per-IP + per-user throttling (see `API Documentation.md` §11).
- **Audit logging:** every mutation → `ActivityLog`.
- **Anomaly alerting:** many failed logins, bulk downloads.
- **CI:** dependency scan (Snyk/Dependabot), secret scan, SAST.

---

## 9. Secrets & Config

- Secrets in AWS Secrets Manager / Doppler / encrypted env; never in code or logs.
- CI uses OIDC (no long-lived keys).
- JWT key rotation via JWKS; KMS for field encryption; least-privilege IAM.

---

## 10. Compliance Readiness

- GDPR: minimize PII, configurable retention, data-residency awareness, DPA with KYC
  vendor.
- SOC 2 Type II + ISO 27001 expectations for enterprise deals.
- KYC/AML: vendor-based; KYB for companies, KYC for individuals.

---

## 11. Security Test Checklist (CI gate)

- [ ] Tenant isolation tests fail cross-tenant reads.
- [ ] IDOR tests denied on `/projects/:id`, `/documents/:id`.
- [ ] Auth: wrong password, replayed token, expired refresh rejected.
- [ ] File upload: malicious MIME / double extension rejected.
- [ ] Rate limit returns 429 with headers.
- [ ] CSP/headers present; no secrets in logs.
- [ ] Dependency + secret scan clean.
