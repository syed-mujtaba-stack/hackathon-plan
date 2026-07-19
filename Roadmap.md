# SoftwPortal — Roadmap

> Long-range product & engineering roadmap from hackathon prototype to funded startup.
> Companion: `PLANNING.md` (two-track plan), `Sprint Planning.md`, `Module-wise Tasks.md`.

---

## Phase 0 — Hackathon (Presentation MVP)
**Goal:** Demo-able slice that proves the idea.
- Auth (Company/Freelancer), KYC submit + Admin approve (stubbed vendor).
- Workspace creation, invite client, project + milestone, document upload (local).
- Socket.IO chat, dark/light, responsive UI.
- *Out of scope:* real KYC vendor, encryption, payments, AI.

## Phase 1 — Validation (Weeks 1–4, post-hackathon)
- Harden auth: Argon2id, RS256, refresh rotation, MFA for admins.
- Real KYC vendor (Persona/Stripe), encrypted doc storage (SSE-KMS).
- Real email (Resend) + OTP; S3/R2 presigned upload + scan.
- Tenant isolation tests in CI; WAF + CSP + rate limiting.
- 5–10 design-partner companies onboard.

## Phase 2 — Collaboration Depth (Weeks 5–10)
- Comments, announcements, @mentions, presence.
- Document version history UI + diff.
- Notification center + preferences.
- Mobile pass + a11y audit.
- Basic analytics for Super Admin.

## Phase 3 — Business & Phase 2 Features (Weeks 11–20)
- Subscriptions (Stripe): Company paid tiers; Freelancer free; Client free→premium.
- Invoicing, contracts, e-signatures.
- Video/voice/screen (WebRTC).
- AI assistant: meeting summary, proposal/contract/estimation generators.
- Public API + integrations (Slack, Google Drive).

## Phase 4 — Scale & Enterprise (Weeks 20+)
- Hybrid tenancy: dedicated schema/DB for enterprise.
- Sharded Redis adapter; Neon autoscaling; read replicas for analytics.
- SOC 2 Type II + ISO 27001; data residency options.
- SSO/SAML for enterprise; audit export; admin roles delegation.

---

## Theme Milestones

| Quarter | Theme | Key Ship |
|---------|-------|----------|
| Hackathon | Prove | Live demo |
| Q+1m | Trust | Real KYC + encryption |
| Q+2-3m | Collaborate | Comments, versions, notifications |
| Q+3-5m | Monetize | Subscriptions + Phase 2 |
| Q+5m+ | Scale | Enterprise tenancy + compliance |

---

## Metrics by Phase

- **Hackathon:** judge feedback, demo completion, clarity of narrative.
- **Validation:** KYC approval rate, workspace activation, invitation→join rate.
- **Collaboration:** DAU/MAU, messages/week, files/week, 4-week retention.
- **Business:** MRR, paid workspace %, churn, expansion.
- **Scale:** p99 latency, socket concurrency, incident MTTR.

---

## Risks to Roadmap

- KYC vendor cost/compliance delays → keep stub path for dev.
- Scope creep → protect hackathon demo scope; phase gates.
- Security regressions → security tests are CI gates, not optional.
- Hiring/velocity → monorepo + contract typing keeps onboarding fast.

---

## Navigation

- **Previous:** [Development Rules.md](./Development%20Rules.md) (how we build).
- **Next read:** [Sprint Planning.md](./Sprint%20Planning.md) — turn this roadmap into sprints.
- **Backlog:** [Module-wise Tasks.md](./Module-wise%20Tasks.md).
- **Master map:** [INDEX.md](./INDEX.md).
