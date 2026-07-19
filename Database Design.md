# SoftwPortal — Database Design

> Schema design for the multi-tenant PostgreSQL database. Enforced with Prisma +
> Row-Level Security (defense-in-depth). Companion: `Architecture.md`, `Security.md`.

---

## 1. Tenant Isolation Strategy

- **Pattern:** shared schema, `workspaceId` UUID column on every tenant-scoped table.
- **RLS:** enabled as secondary control; `SET LOCAL app.workspace_id` per transaction.
- **Composite uniques:** `(workspaceId, email)`, `(workspaceId, slug)` prevent leaks.
- **FKs:** tenant-scoped rows reference `Workspace(id)`.

---

## 2. Entity Relationship Overview

```
User 1—N Membership N—1 Workspace
User 1—N KycSubmission
Workspace 1—N Project 1—N Milestone
Workspace 1—N Document
Workspace 1—N Message
Workspace 1—N Invitation
User 1—N Session
Workspace 1—N ActivityLog
User 1—N Notification
```

---

## 3. Tables (key fields)

### User
| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | `gen_random_uuid()` |
| email | citext UNIQUE | lowercased |
| passwordHash | text | Argon2id |
| phone | text | E.164 |
| role | enum | SUPER_ADMIN, COMPANY, FREELANCER, CLIENT |
| verificationStatus | enum | UNVERIFIED, PENDING, VERIFIED, REJECTED |
| kycStatus | enum | NOT_SUBMITTED, PENDING, VERIFIED, REJECTED |
| mfaSecret | text | AES-256-GCM encrypted (nullable) |
| mfaEnabled | bool | default false |
| createdAt / updatedAt | timestamptz | |

### Workspace
| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| ownerId | UUID FK → User | |
| type | enum | COMPANY, FREELANCER |
| name | text | |
| slug | text | unique per tenant context |
| plan | enum | FREE, STARTER, PRO (default FREE) |
| createdAt | timestamptz | |

### Membership
| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| workspaceId | UUID FK → Workspace | tenant key |
| userId | UUID FK → User | |
| roleInWorkspace | enum | OWNER, MEMBER, CLIENT |
| invitedBy | UUID FK → User | nullable |
| status | enum | INVITED, ACTIVE, REVOKED |

### Invitation
| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| workspaceId | UUID FK | |
| token | text UNIQUE | `crypto.randomBytes(32)` hex |
| email | citext | bound email |
| role | enum | |
| expiresAt | timestamptz | TTL (e.g., +7d) |
| usedAt | timestamptz | single-use |

### KycSubmission
| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| userId | UUID FK | |
| type | enum | COMPANY, FREELANCER, CLIENT |
| docs | jsonb | array of encrypted file keys + metadata |
| status | enum | PENDING, VERIFIED, REJECTED |
| reviewedBy | UUID FK → User | Super Admin |
| reviewedAt | timestamptz | |
| note | text | rejection reason |

### Project
| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| workspaceId | UUID FK | tenant key |
| title | text | |
| status | enum | PLANNING, ACTIVE, ON_HOLD, DONE |
| priority | enum | LOW, MEDIUM, HIGH, URGENT |
| timelineStart / timelineEnd | date | |
| clientId | UUID FK → User | nullable |
| createdBy | UUID FK → User | |

### Milestone
| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| projectId | UUID FK → Project | |
| title | text | |
| dueDate | date | |
| status | enum | PENDING, APPROVED |

### Document
| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| workspaceId | UUID FK | tenant key |
| projectId | UUID FK → Project | nullable |
| uploadedBy | UUID FK → User | |
| fileKey | text | UUID key in storage |
| originalName | text | display only |
| mime | text | verified magic-byte MIME |
| size | bigint | bytes |
| version | int | current version |
| versionHistory | jsonb | [{version, fileKey, uploadedBy, at}] |

### Message
| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| workspaceId | UUID FK | tenant key |
| projectId | UUID FK | nullable (workspace chat) |
| senderId | UUID FK → User | |
| body | text | sanitized |
| createdAt | timestamptz | |

### ActivityLog
| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| workspaceId | UUID FK | tenant key |
| actorId | UUID FK → User | |
| action | text | e.g., PROJECT_CREATED |
| entityType | text | |
| entityId | UUID | |
| createdAt | timestamptz | |

### Notification
| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| userId | UUID FK | (user may belong to many workspaces) |
| type | enum | MESSAGE, MILESTONE, INVITE, KYC |
| payload | jsonb | |
| readAt | timestamptz | nullable |

### Session
| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| userId | UUID FK | |
| refreshTokenHash | text | SHA-256 of refresh token |
| family | UUID | token family for rotation/revocation |
| device | text | UA |
| ip | inet | |
| expiresAt | timestamptz | |
| revokedAt | timestamptz | nullable |

---

## 4. Indexes

- `User(email)`, `User(workspaceId)` (via membership), `User(role)`
- `Workspace(ownerId)`, `Workspace(slug)`
- `Membership(workspaceId, userId)` unique
- `Project(workspaceId, status)`
- `Document(workspaceId, projectId)`
- `Message(workspaceId, projectId, createdAt)`
- `ActivityLog(workspaceId, createdAt)`
- `Invitation(token)`, `Invitation(workspaceId, email)`

---

## 5. Row-Level Security (sketch)

```sql
ALTER TABLE "Project" ENABLE ROW LEVEL SECURITY;
CREATE POLICY project_tenant ON "Project"
  USING (workspace_id = current_setting('app.workspace_id')::uuid)
  WITH CHECK (workspace_id = current_setting('app.workspace_id')::uuid);
ALTER TABLE "Project" FORCE ROW LEVEL SECURITY;
```

Apply equivalent policies to every tenant-scoped table. Tests use `FORCE ROW LEVEL
SECURITY` so superuser cannot mask a missing filter.

---

## 6. Migrations & Conventions

- Single source: `packages/db` Prisma schema.
- Migrations run **before** rolling deploy (zero-downtime).
- Never rename columns in place; use additive + backfill + drop.
- Seed: one Super Admin, sample workspace for demos.
- Backups: Neon PITR; per-tenant export possible via `workspaceId` filter.
