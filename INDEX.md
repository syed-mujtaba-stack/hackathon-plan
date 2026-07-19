# SoftwPortal — Documentation Index

> **START HERE.** This is the master map of all SoftwPortal docs. Pick your role and
> read top-to-bottom. Every doc also has a "Navigation" footer linking to the next
> file, so you never have to guess what to open.

---

## 📖 How to Read (Step by Step)

Read in this order — each file builds on the previous:

1. **[README.md](./README.md)** — *What is SoftwPortal?* The product (PRD): vision,
   users, roles, features, MVP goals. **Start here if you know nothing.**
2. **[PLANNING.md](./PLANNING.md)** — *How we'll build it in two tracks.* Hackathon
   (demo MVP) vs Startup (production). Includes the security summary.
3. **[RESEARCH.md](./RESEARCH.md)** — *Deep technical research.* Multi-tenancy, auth,
   KYC, realtime, files, security, architecture choices. **The "why" behind decisions.**
4. **[Architecture.md](./Architecture.md)** — *System design.* Topology, monorepo,
   layers, request lifecycle, scaling.
5. **[Database Design.md](./Database%20Design.md)** — *Data model.* Tables, tenant
   isolation, RLS, indexes.
6. **[API Documentation.md](./API%20Documentation.md)** — *Endpoints + realtime.*
   Every REST route and Socket.IO event.
7. **[UI-UX.md](./UI-UX.md)** — *Interface.* Design tokens, screens, components.
   (Note: old `UI/UX.md` is a redirect — ignore it.)
8. **[Security.md](./Security.md)** — *Defense.* Full control set + CI test checklist.
9. **[Development Rules.md](./Development%20Rules.md)** — *How we code.* Conventions,
   Definition of Done.
10. **[Roadmap.md](./Roadmap.md)** — *Where we're going.* Phase 0→4.
11. **[Sprint Planning.md](./Sprint%20Planning.md)** — *How we work.* Hackathon blocks
    + 2-week sprints.
12. **[Module-wise Tasks.md](./Module-wise%20Tasks.md)** — *What to build.* Granular
    backlog by module.

---

## 🧭 By Role (jump to what matters to you)

| You are… | Read these, in order |
|----------|----------------------|
| **Founder / PM** | README → PLANNING → Roadmap → Sprint Planning |
| **New Developer** | README → PLANNING → Architecture → Development Rules → Module-wise Tasks |
| **Backend Dev** | Architecture → Database Design → API Documentation → Security → Development Rules |
| **Frontend Dev** | UI-UX → API Documentation → Architecture → Development Rules |
| **Security / Reviewer** | Security → RESEARCH → Architecture → Database Design |
| **Designer** | UI-UX → README (features) → PLANNING (demo scope) |
| **Hackathon Team** | README → PLANNING (§4.1) → Module-wise Tasks (H items) → Sprint Planning (B1–B6) |

---

## 🗺️ Visual Map

```
README (PRD)
  └─▶ PLANNING (two-track + security summary)
        └─▶ RESEARCH (deep technical "why")
              ├─▶ Architecture ──▶ Database Design
              │                    └─▶ API Documentation
              ├─▶ UI-UX
              ├─▶ Security
              ├─▶ Development Rules
              └─▶ Roadmap ──▶ Sprint Planning ──▶ Module-wise Tasks
```

---

## ✅ Quick Checklist for the Team

- [ ] Everyone reads README first
- [ ] Hackathon team follows Sprint Planning B1–B6
- [ ] Build order = Module-wise Tasks dependency order
- [ ] Security checklist in Security.md is a CI gate
- [ ] Keep docs in sync (Development Rules §8)

---

*Repo: github.com/syed-mujtaba-stack/hackathon-plan*
