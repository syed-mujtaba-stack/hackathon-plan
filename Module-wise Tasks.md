# SoftwPortal — Module-wise Tasks

> Granular task breakdown by module. Maps to `Sprint Planning.md` and `Roadmap.md`.
> Use as the backlog. [H] = hackathon, [S] = startup track.

---

## Module 1 — Auth [H][S]
- [H] Register (Company/Freelancer) with role selection
- [H] Login + JWT session (access in memory, refresh cookie)
- [H] Email verification link (stub send)
- [S] Argon2id hashing + HIBP breach check
- [S] RS256 JWT + refresh rotation + family revocation
- [S] MFA (TOTP + WebAuthn for admins)
- [S] Password reset (signed token)
- [S] Account lockout + rate limit + generic errors
- [S] Session table + logout-all / per-device revoke

## Module 2 — KYC & Verification [H][S]
- [H] KYC submission form (company docs, freelancer ID)
- [H] Admin KYC queue + approve/reject (stub)
- [S] Integrate Persona/Stripe vendor
- [S] Encrypted doc storage (SSE-KMS) + quarantine→scan→promote
- [S] KYB for companies, KYC for individuals
- [S] Audit log of all review actions

## Module 3 — Workspaces & Invitations [H][S]
- [H] Workspace auto-create on verification
- [H] Generate invite link `/invite/{token}`
- [H] Client accepts invite → joins workspace
- [S] TenantContext middleware + RLS `SET LOCAL`
- [S] Revoke invitation; TTL enforcement
- [S] Workspace switcher (multi-membership users)
- [S] Role in workspace (OWNER/MEMBER/CLIENT)

## Module 4 — Projects [H][S]
- [H] Create project (title, status, priority, timeline)
- [H] Project board list + detail
- [H] Milestones (create, due date)
- [S] Team assignment + client link
- [S] Milestone approval by client + notification
- [S] Status workflow + activity logging
- [S] Search/filter/sort

## Module 5 — Documents [H][S]
- [H] Upload (local disk) + list + download
- [S] Presigned S3/R2 upload (direct)
- [S] MIME allow-list + magic-byte verification
- [S] Malware scan (quarantine→promote)
- [S] Signed download URL (short TTL)
- [S] Version history + preview

## Module 6 — Communication (Socket.IO) [H][S]
- [H] Real-time project/workspace chat
- [H] Presence (online dots)
- [H] Typing indicator
- [S] Redis sharded adapter (horizontal scale)
- [S] Notification center (in-app + email)
- [S] Announcements (admin→workspace)
- [S] Reconnection + state recovery

## Module 7 — Admin & Dashboards [H][S]
- [H] Separate dashboards (Company/Freelancer/Client/Admin)
- [H] Admin KYC queue
- [S] User/company management
- [S] Reports + analytics (Super Admin)
- [S] Platform settings + security monitoring
- [S] Dispute resolution workflow

## Module 8 — Activity & Profiles [H][S]
- [H] Activity timeline (basic events)
- [H] User profile (photo, contact, badge)
- [S] Full activity log + filters
- [S] Social links, company details
- [S] Verification badges everywhere

## Module 9 — Security & Infra [S]
- [S] WAF + CSP + security headers
- [S] Rate limiting (per-IP + per-user)
- [S] Secret management (Secrets Manager/Doppler) + OIDC CI
- [S] CI: lint/type/test + dep/secret scan
- [S] Pre-deploy migration task
- [S] Monitoring/alerting + health endpoints
- [S] Tenant isolation + IDOR test suite (CI gate)

## Module 10 — Phase 2 (later) [ ]
- [ ] Video/voice/screen (WebRTC)
- [ ] Calendar
- [ ] Invoicing, contracts, e-signatures
- [ ] Payments (Stripe)
- [ ] AI assistant (summary/proposal/contract/estimation)
- [ ] Public API + integrations

---

## Dependency Order

1. Auth → 2. KYC → 3. Workspaces → 4. Projects → 5. Documents → 6. Chat →
7. Admin → 8. Activity → 9. Security/Infra (parallel from Phase 1).

Hackathon path: 1 → 2(stub) → 3 → 4 → 5(local) → 6 → 7(stub) → 8(basic).
