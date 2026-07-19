# SoftwPortal — Sprint Planning

> How work is planned and executed. Two modes: **Hackathon sprint** (speed) and
> **Startup sprints** (2-week Scrum). Companion: `Roadmap.md`, `Module-wise Tasks.md`.

---

## 1. Hackathon Sprint (24–48h)

Single time-boxed sprint. Goal: working demo. Strict scope; stub the rest.

| Block | Hours | Deliverable | Owner |
|-------|-------|-------------|-------|
| B1 Setup | 0–4 | Monorepo, Next.js + Postgres + Prisma, auth | BE |
| B2 Verify | 4–10 | Workspace + KYC submit + Admin approve | BE |
| B3 Collaborate | 10–16 | Invite client + project + document upload | FE+BE |
| B4 Realtime | 16–22 | Socket.IO chat + notifications | BE |
| B5 Polish | 22–28 | UI, dark/light, responsive, rehearsal | FE |
| B6 Buffer | 28–32 | Recording, README, contingency | All |

Daily (or per-block) sync; demo rehearsal in B5. Definition of Done = runs live
without errors for the scripted demo.

---

## 2. Startup Sprints (2 weeks)

- **Cadence:** 2-week Scrum. Sprint planning (Mon), mid-sprint review, demo + retro.
- **Backlog:** groomed from `Roadmap.md` and `Module-wise Tasks.md`.
- **Refinement:** tickets sized (S/M/L), acceptance criteria, security checklist.
- **WIP limit:** 2–3 active tickets per dev to avoid context switch.
- **CI gate:** PR must pass lint, typecheck, tests (incl. tenant isolation + IDOR).
- **Demo:** stakeholder review; shippable to staging; prod on approval.

---

## 3. Ticket Template

```
Title: <module> — <action>
Module: Auth | KYC | Workspace | Project | Document | Chat | Admin | Infra
Type: feat | fix | security | docs | chore
Acceptance:
  - [ ] ...
Security: (if applicable) tenant scope, IDOR, validation, rate limit
Tests: unit / integration / security
Links: API doc, design
```

---

## 4. Prioritization

1. Security-blocking (auth, tenant isolation, file safety).
2. Core collaboration (projects, chat, docs).
3. Verification (KYC admin).
4. Polish (UI/UX, notifications).
5. Nice-to-have (Phase 2 features).

---

## 5. Ceremonies & Tools

- Board: GitHub Projects (or Linear). Columns: Backlog, Ready, In Progress, Review,
  Done.
- Labels: `security`, `hackathon`, `bug`, `tech-debt`, `phase-2`.
- Retro actions tracked as tickets; no action = no learning.
- On-call: post-launch, rotating; incidents logged to `ActivityLog` + postmortem.

---

## 6. Definition of Done (recap)

Code reviewed, tests green (incl. security), docs updated, CI clean, demoed in
staging, no new secrets.

---

## Navigation

- **Previous:** [Roadmap.md](./Roadmap.md) (the phases this sprinting serves).
- **Next read:** [Module-wise Tasks.md](./Module-wise%20Tasks.md) — the actual backlog to pull into sprints.
- **Rules:** [Development Rules.md](./Development%20Rules.md).
- **Master map:** [INDEX.md](./INDEX.md).
