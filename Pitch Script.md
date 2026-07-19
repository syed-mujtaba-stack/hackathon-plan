# SoftwPortal — Pitch Script (Hackathon)

> A 30-second, 2-minute, and 5-minute version. Use the 30s if the timer is tight;
> expand to 2–5 min for Q&A and demo. Keep the problem→solution→proof structure.

---

## 30-Second Version (elevator pitch)

"Professional work is fragmented and unverified. Companies chat on WhatsApp, clients
reply on email, files live in Drive, and nobody is actually verified — so scams,
leaked IP, and unresolved disputes are the norm. SoftwPortal fixes this with **one
verified workspace** per business relationship: you get KYC-checked companies and
freelancers, clients brought in only through secure invitations, and a single place
for chat, documents, projects, and an audit trail — all with bank-grade isolation.
We're the secure operating system for trusted business collaboration, not another
marketplace that owns your client."

---

## 2-Minute Version

**Problem (30s).** "Every company–client engagement today is spread across five apps —
WhatsApp, email, Drive, a task tool, a spreadsheet. Nobody is verified, files leak by
default, and there's no audit trail when something goes wrong. Upwork and Fiverr are
discovery marketplaces; they don't give teams that already have clients a *secure place
to actually work*."

**Solution (45s).** "SoftwPortal is one verified, isolated workspace per relationship.
Companies and freelancers pass KYC/KYB before they can even create a workspace. Clients
join only through a single-use, expiring invite. Inside: chat, documents, project boards,
milestones, and a tamper-evident activity log — no tool hopping, no lost context.
Every workspace is cryptographically isolated, so a client only ever sees their own work."

**Proof / Demo (30s).** "In our demo: a company registers and gets KYC-verified by an
admin, invites a client through a secure link, creates a project with milestones, uploads
a document over an encrypted signed URL, and chats in real time. All of it runs on
Argon2id, RS256, MFA, and OWASP-API-Top-10 defenses from day one."

**Ask / Close (15s).** "We're building the trusted layer the freelance and agency
economy is missing. Try the demo, or talk to us about design-partner onboarding."

---

## 5-Minute Version (with walkthrough)

1. **Cold open (30s):** the fragmented-tooling pain, told as a story — "Imagine your
   agency onboarding a $20k client over WhatsApp, Drive, and email, and the file gets
   forwarded to the wrong person."
2. **Problem deep-dive (60s):** no identity assurance, data leaks by default, no audit
   trail, marketplaces don't fit. Reference the five-point problem in `README.md`.
3. **Solution deep-dive (90s):** verify-first model, one workspace, hard isolation,
   provable trust, secure-by-default. Walk the architecture diagram.
4. **Live demo (90s):** register company → admin KYC approve → invite client → project
   + milestone → document upload (signed URL) → real-time chat + presence.
5. **Why we win (30s):** not a marketplace; verification + isolation are the moat;
   security is built in, not bolted on.
6. **Roadmap & ask (30s):** subscriptions, AI assistant, enterprise tenancy; invite
   judges to be design partners.

---

## Demo Script (scripted clicks)

1. Open landing page → "Secure workspace for verified companies, freelancers & clients."
2. Register as **Company** → submit KYC docs.
3. Switch to **Admin** → open KYC queue → **Approve**.
4. Back to Company → copy `/invite/{token}` → open in private window as **Client** → join.
5. Create **Project** "Website Redesign" + add **Milestone** with due date.
6. Upload a **Document** → show signed-download flow.
7. Send a **chat message** → show real-time delivery + presence dot.
8. Close with: "Every action above is isolated per workspace and written to an audit log."

---

## Anticipated Questions

- **"Isn't this just Upwork?"** — No. We're a private workspace for teams that already
  have clients; we don't take a marketplace cut or own the relationship.
- **"How is data isolated?"** — Per-workspace tenant keys + PostgreSQL row-level
  security; cross-tenant reads fail in CI tests.
- **"What about payments?"** — Phase 3 (Stripe subscriptions); hackathon proves the
  trust layer first.
- **"Is the KYC real?"** — Stubbed for the demo (admin approve); Persona/Stripe planned
  for validation phase.
