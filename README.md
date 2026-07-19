# Product Requirements Document (PRD)

# SoftwPortal

### A Secure Workspace for Verified Companies, Freelancers & Clients

**Version:** 1.0 (MVP)
**Owner:** SoftwDocs
**Product Type:** Multi-Tenant SaaS Platform
**Last updated:** 2026-07-19

---

# Vision

SoftwPortal is a secure, modern SaaS platform where **verified companies, verified
freelancers, and verified clients** collaborate in dedicated, isolated workspaces.

Unlike freelance marketplaces, SoftwPortal is a **private collaboration platform**.
Every company or freelancer invites their own clients into a secure workspace where
they can communicate, share documents, manage projects, and track work.

The platform focuses on **trust, verification, security, and collaboration**.

---

# Problem & Our Solution

## The Problem (Detail)

The professional services world runs on **fragmented, unverified, and insecure tooling**.
A single company–client engagement today is spread across five or more disconnected
apps, and at no point is either party guaranteed to be who they claim to be.

1. **No single source of truth.** Companies chat on WhatsApp, clients reply on Email,
   documents live in Google Drive, tasks sit in Trello/Notion, and invoices are stored
   in a spreadsheet. Context is lost every time the conversation jumps a tool.
2. **Zero identity assurance.** Anyone can claim to be an "agency" or a "client." There
   is no shared, trusted verification layer, so scams, ghosting, and fake deliverables
   are common — and expensive.
3. **Data leaks by default.** Files and messages are shared through personal accounts
   with no tenant boundaries. One wrong forward exposes every client's confidential work.
4. **No audit trail.** When a milestone is approved or a document changes, there is no
   tamper-evident record, making disputes slow and unprovable.
5. **Marketplaces don't fit.** Upwork/Fiverr own the relationship and take a cut; they
   are discovery engines, not private workspaces. Teams that already have their clients
   still have nowhere secure to actually do the work.

The result: wasted hours, leaked IP, unresolved disputes, and a persistent trust gap
between the people trying to do business together.

## Our Solution (Detail)

SoftwPortal replaces that chaos with **one verified, isolated workspace per
business relationship**.

- **Verify first, collaborate second.** Every company and freelancer passes KYC/KYB
  (admin-reviewed, encrypted documents) before they can create a workspace. Clients are
  brought in only through single-use, expiring, unguessable invitations — so every
  participant is a real, attributable identity.
- **One trusted workspace.** Once verified, the same workspace holds the chat, the
  documents, the project board, the milestones, and the activity log — no more tool
  hopping, no lost context.
- **Hard isolation by design.** Cryptographic tenant separation (per-workspace keys +
  row-level security) means a client can only ever see the work they were invited to.
  No accidental cross-client leaks.
- **Provable trust.** Every mutation writes to an append-only activity log, so milestone
  approvals and document changes are auditable and dispute-ready.
- **Secure by default, not by add-on.** Argon2id hashing, RS256 JWTs, rotating refresh
  tokens, MFA for admins, presigned/signed file access, and OWASP-API-Top-10 defenses
  are built in from day one — not bolted on after a breach.

**In one line:** SoftwPortal is the secure operating system for verified business
collaboration — where companies, freelancers, and clients finally work in one trusted
place instead of scattering across unverified apps.

---

# Problem Statement

Today's workflow is fragmented.

- Companies use WhatsApp.
- Clients use Email.
- Documents are scattered across Google Drive.
- Projects are managed somewhere else.
- Invoices are stored separately.

This creates:

- Poor communication
- Lost documents
- Security risks
- Fake identities
- Difficult project tracking
- No centralized, trusted workspace

SoftwPortal solves this problem.

---

# Mission

Build the most trusted, secure collaboration platform for companies, freelancers,
and clients — where every participant is verified and every interaction is protected.

---

# Target Users

## Companies
Software houses, marketing agencies, design agencies, architecture firms, law firms,
construction companies, accounting firms.

## Freelancers
Developers, designers, video editors, writers, consultants, marketing experts.

## Clients
Any business or individual invited by a company or freelancer. Clients never register
directly; they join only through a verifiable invitation.

---

# User Roles

## Super Admin
Controls the entire platform: KYC approval, user/company/freelancer/client management,
reports, analytics, platform settings, security monitoring, dispute resolution.

## Company
Create workspace, invite clients, invite employees, create/manage projects, upload
files, assign work, communicate, view reports.

## Freelancer (Free Account)
Create workspace, invite own clients, manage projects, upload files, communicate,
deliver work.

## Client
Join via invitation, create project requests, upload requirements, chat, approve
milestones, download files, track progress.

---

# Registration & Verification Flow

## Company Registration
Submits: company name, website, business email, phone, country, company registration
number, government license, business certificate, owner identity, address proof.
Flow: **Pending KYC → Admin Review → Verified → Workspace Created**.

## Freelancer Registration
Submits: name, government ID, email, phone, skills, portfolio (optional).
Flow: **Pending Verification → Verified → Workspace Created**.

## Client Registration
Client never registers directly. Receives invitation (e.g. `portal.com/invite/ABC123`)
→ Registers → Verification → Workspace Access (scoped to inviter only).

---

# Core Features (MVP)

## Authentication
Email login, password login, email verification, OTP verification, password reset.

## KYC & Verification
Companies, freelancers, and clients require admin-approved verification. Documents are
stored encrypted; review is audited.

## Multi-Tenant Workspaces
Every company and freelancer gets an isolated workspace. Cryptographic tenant
separation guarantees no data leakage between workspaces.

## Dashboard
Separate dashboards for Company, Freelancer, Client, and Admin.

## Client Invitation System
Companies and freelancers generate single-use, expiring, unguessable invitation links.
Clients can access only the inviter's workspace.

## Projects
Create project, status, priority, timeline, team members, client information,
milestones, deadlines.

## Document Management
Upload PDF, images, ZIP, Word, Excel, videos. Preview, download (via signed URLs),
version history.

## Communication
Real-time chat (Socket.IO), project comments, announcements.

## Notifications
Email, in-app, and real-time.

## Activity Timeline
Every action recorded: project created, document uploaded, message sent, status changed,
milestone approved.

## User Profiles
Profile photo, contact details, verification status, company details, social links.

---

# Security (Overview)

SoftwPortal is security-first. All communications are encrypted in transit (TLS 1.2+),
credentials are never stored in plaintext, sessions are short-lived and revocable,
and every workspace is logically isolated. A full threat model and defensive controls
are specified in `PLANNING.md` (Security section).

Key controls: HTTPS/TLS, JWT (access + refresh), RBAC, audit logs, session/device
tracking, rate limiting, encrypted storage, signed file access, tenant isolation,
input validation, and protection against common web attacks (OWASP Top 10).

---

# Future Features (Phase 2)

Video meetings, voice calls, screen sharing, calendar, invoicing, contracts,
e-signatures, payments, AI assistant, AI meeting summary, AI proposal/contract/estimation
generators.

---

# Revenue Model

- **Companies:** Paid subscription.
- **Freelancers:** Free.
- **Clients:** Initially free; premium features in future.

---

# Success Metrics

Registered companies, freelancers, clients; verified accounts; active workspaces;
projects created; monthly active users; file uploads; client invitations; workspace
retention. (Refined funnel metrics in `PLANNING.md`.)

---

# Design Principles

Modern, enterprise, fast, secure, minimal, mobile-first, dark & light mode, responsive,
accessible.

---

# MVP Goals

The first release must focus on five core capabilities:

1. Secure Registration & Authentication
2. Admin KYC Verification
3. Multi-Tenant Workspaces
4. Project & Document Management
5. Secure Communication

Everything else is added after validating the product with real users.

---

# Product Vision

SoftwPortal aims to become the operating system for secure business collaboration —
one verified workspace replacing fragmented communication, document sharing, project
management, and client onboarding tools. It is the flagship collaboration product
within the SoftwDocs ecosystem.

---

## Navigation

- **Next read:** [PLANNING.md](./PLANNING.md) — how we build it (hackathon + startup tracks).
- **Deep dive:** [RESEARCH.md](./RESEARCH.md) — technical research behind decisions.
- **Pitch:** [Pitch Script.md](./Pitch%20Script.md) — 30s/2min/5min pitch + demo script.
- **Lost?** Start from [INDEX.md](./INDEX.md) (master map by role).
