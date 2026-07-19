# SoftwPortal — API Documentation

> REST + realtime API contract. Base URL: `https://api.softwportal.com` (web:
> `NEXT_PUBLIC_API_URL`). All timestamps ISO-8601. All requests JSON; all responses
> `application/json`. Companion: `Architecture.md`, `Security.md`.

---

## 1. Conventions

- **Auth header:** `Authorization: Bearer <access_token>`.
- **Tenant scoping:** implicit via JWT `workspaceId`; never pass as a param.
- **Errors:** `{ "error": "CODE", "message": "..." }` with proper HTTP status.
- **Validation:** 422 on DTO failure; 401 unauthorized; 403 forbidden; 404 not found
  (generic); 429 rate limited.
- **Pagination:** `?page=&limit=`; response `{ data, page, limit, total }`.

---

## 2. Authentication

### POST /auth/register
Register Company/Freelancer.
```json
{ "email":"a@b.com", "password":"...", "role":"COMPANY",
  "company": { "name":"Acme", "website":"https://acme.com", "country":"PK",
               "regNumber":"...", "phone":"+...", "addressProof":"<fileKey>" } }
```
→ `201` → status PENDING_KYC.

### POST /auth/login
```json
{ "email":"a@b.com", "password":"..." }
```
→ `200` `{ accessToken, expiresIn }` + `refreshToken` HttpOnly cookie.
If MFA enabled → returns `{ mfaRequired:true, challengeToken }`; then
`POST /auth/mfa/verify { challengeToken, code }`.

### POST /auth/refresh
Uses cookie → rotates refresh, returns new `accessToken`.

### POST /auth/logout
Revokes current session (family).

### POST /auth/forgot-password / reset-password
Signed single-use token, short TTL.

### POST /auth/verify-email
Email verification link handler.

---

## 3. KYC

### POST /kyc/submit
Upload docs (file keys from presigned upload). `{ type, docs:[{kind,fileKey}] }`.
→ status PENDING.

### GET /admin/kyc/queue
Super Admin only. List PENDING submissions.

### POST /admin/kyc/:id/approve | /reject
Super Admin only. Approve → workspace active; reject → `{ note }`.

---

## 4. Workspaces & Invitations

### GET /workspaces/me
Current workspace + role + verification status.

### POST /invitations
Generate invite. `{ email, role }` → `{ token, inviteUrl }`.
`inviteUrl = https://app.softwportal.com/invite/{token}`.

### POST /invite/:token/accept
Client registers/accepts → joins workspace.

### DELETE /invitations/:id
Revoke (single-use already enforced).

---

## 5. Projects

### GET /projects
List (scoped to workspace). Filters: `?status=&priority=`.

### POST /projects
```json
{ "title":"Website Redesign", "priority":"HIGH",
  "timelineStart":"2026-08-01", "timelineEnd":"2026-09-15",
  "clientId":"<uuid>", "team":["<userId>"] }
```

### GET /projects/:id
Includes milestones, team, client.

### PATCH /projects/:id
Update fields (mass-assignment protected by DTO allow-list).

### POST /projects/:id/milestones
`{ "title", "dueDate" }` → status PENDING.

### POST /projects/:id/milestones/:mid/approve
Client approval → status APPROVED + notification.

---

## 6. Documents

### POST /documents/presign
`{ fileName, mime, size, projectId? }` → `{ uploadUrl (PUT, 120s), fileKey }`.
Client uploads directly to S3/R2, then:

### POST /documents/finalize
`{ fileKey }` → creates Document (after scan).

### GET /documents
List scoped to workspace/project.

### GET /documents/:id/download
Returns short-lived signed URL (60–300s).

### GET /documents/:id/versions
Version history.

---

## 7. Chat (REST side)

### GET /projects/:id/messages?page=
History (scoped). Live messages arrive over Socket.IO.

---

## 8. Activity & Notifications

### GET /activity
Timeline for workspace.

### GET /notifications
`{ data, unreadCount }`.

### POST /notifications/:id/read
Mark read.

---

## 9. Real-Time (Socket.IO)

Connect: `wss://api.softwportal.com` with JWT auth.

### Client → Server
- `join` `{ scope:"workspace"|"project", id }`
- `message:send` `{ projectId?, body }`
- `typing` `{ projectId? }`
- `presence:ping`

### Server → Client
- `message:new` `{ id, senderId, body, at }`
- `notification:new` `{ type, payload }`
- `presence:update` `{ userId, online }`
- `typing` `{ userId }`

Rooms: `workspace:{id}`, `project:{id}`. Auth failure → disconnect.

---

## 10. Admin

### GET /admin/users, /admin/companies, /admin/metrics
Super Admin only. Reports + analytics.

---

## 11. Rate Limits

| Endpoint group | Limit |
|----------------|-------|
| /auth/* | 5 / 15 min per IP+account |
| /invitations | 20 / hour |
| /documents/presign | 60 / min |
| Others | 100 / min per user |

Returns `429` + `Retry-After` + `X-RateLimit-*`.
