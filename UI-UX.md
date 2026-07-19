# SoftwPortal — UI/UX Specification

> Design language, information architecture, and component guidance for the SoftwPortal
> web client (Next.js + Tailwind + shadcn/ui). Companion: `Architecture.md`,
> `Module-wise Tasks.md`.

---

## 1. Design Principles (from PRD)

Modern, enterprise, fast, secure, minimal, mobile-first, dark & light mode,
responsive, accessible (WCAG 2.1 AA).

---

## 2. Design Tokens

- **Color:** neutral slate base; primary brand (indigo/blue); success (emerald),
  warning (amber), danger (rose). All as CSS variables for theme switching.
- **Typography:** Inter for UI, tabular numerals for metrics.
- **Radius:** 8–12px cards; 6px inputs.
- **Spacing:** 4px scale (4/8/12/16/24/32).
- **Dark/Light:** `next-themes`; respects system preference; toggle in top bar.
- **Motion:** subtle (150–200ms), never blocking.

---

## 3. Layout Shell

```
┌──────────────────────────────────────────────┐
│ TopBar: logo | workspace switcher | search |  │
│         notifications | theme toggle | avatar  │
├──────────┬───────────────────────────────────┤
│ Sidebar  │  Main Content (route)              │
│ - Home   │                                    │
│ - Projects│                                   │
│ - Documents│                                  │
│ - Chat   │                                    │
│ - Clients│                                    │
│ - Activity│                                   │
│ (Admin:  │                                    │
│  KYC,    │                                    │
│  Users)  │                                    │
└──────────┴───────────────────────────────────┘
```

- **Mobile:** sidebar collapses to bottom nav / drawer.
- **Roles:** nav items filtered by RBAC (client sees fewer than company).

---

## 4. Key Screens

| Screen | Audience | Purpose |
|--------|----------|---------|
| Landing | Public | Value prop, problem statement, CTA |
| Login / Register | All | Auth + role selection |
| KYC Submission | Company/Freelancer | Upload verification docs |
| Workspace Home | All | Overview: active projects, mentions |
| Project Board | All | Create/manage projects, milestones |
| Document Center | All | Upload, preview, version history |
| Chat | All | Real-time project/workspace chat |
| Invite Client | Company/Freelancer | Generate `/invite/{token}` |
| Admin KYC Queue | Super Admin | Approve/reject submissions |
| Profile | All | Photo, contact, verification badge |

---

## 5. Component Library (shadcn/ui)

- Primitives: Button, Input, Dialog, Sheet, Dropdown, Table, Tabs, Avatar, Badge,
  Toast, Tooltip, Command (palette).
- Domain components: `ProjectCard`, `MilestoneList`, `DocList`, `ChatWindow`,
  `InviteDialog`, `KycUploader`, `VerificationBadge`, `WorkspaceSwitcher`.
- Forms: `react-hook-form` + `zod` resolver; inline field errors.

---

## 6. States & Feedback

- **Loading:** skeletons, not spinners, for lists.
- **Empty:** illustrated empty states with actionable CTA.
- **Error:** inline + toast; never raw API errors.
- **Success:** toast + optimistic UI for chat/messages.
- **Verification:** always show `VerificationBadge` (verified/unverified/pending).

---

## 7. Real-Time UX

- Chat updates live without refresh; typing indicator; unread badges.
- Presence dots on project members.
- Toast for `notification:new` (milestone approved, invited, etc.).

---

## 8. Accessibility

- Keyboard navigation for all flows; focus rings visible.
- Color contrast AA; don't rely on color alone (icons + text).
- Semantic HTML + ARIA for dialogs, live regions (chat, toasts).
- Reduced-motion support.

---

## 9. Responsive Breakpoints

| Breakpoint | Layout |
|-----------|--------|
| <640px | Single column, bottom nav |
| 640–1024px | Collapsible sidebar |
| >1024px | Full shell |

---

## 10. Demo Polish (Hackathon)

- Pre-filled demo accounts (Company, Freelancer, Client, Admin) for instant switching.
- One-click "Admin approve" to fast-forward KYC during the pitch.
- Smooth dark/light transition as a visual hook.

---

## Navigation

- **Previous:** [API Documentation.md](./API%20Documentation.md) (endpoints this UI calls).
- **Next read:** [Security.md](./Security.md) — security requirements the UI must respect.
- **Build it:** [Module-wise Tasks.md](./Module-wise%20Tasks.md) (UI items under each module).
- **Master map:** [INDEX.md](./INDEX.md).
