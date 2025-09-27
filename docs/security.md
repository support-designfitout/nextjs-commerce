# Security & Access Control

Apps: designfitout.com (public site) & fitoutlab.app (authenticated app)
Stack: Cloudflare Workers/Pages, Hyperdrive → PlanetScale (DB), R2, KV, Durable Objects, Queues, Firebase Auth, Cloudflare Zero-Trust.

## 1) Roles & Capabilities

| Role     | Projects | Assets (Drawings/Renders) | BOQ / P&L             | Materials & RFQ    | Supplier Quotes            | Admin/Owner Ops                |
|----------|----------|---------------------------|-----------------------|--------------------|----------------------------|--------------------------------|
| Owner    | C/R/U/D  | C/R/U/D                   | C/R/U/D               | C/R/U/D            | R                          | All (flags, schema, exports)   |
| Designer | C/R/U    | C/R/U                     | C/R/U                 | C/R/U              | R                          | —                              |
| Client   | R        | R (approved only)         | R (snapshots/summary) | R (approved items) | —                          | —                              |
| Supplier | —        | —                         | —                     | R (RFQ assigned)   | C (submit quote), R (own)  | —                              |

**Notes:**
- C=create, R=read, U=update, D=delete
- Client sees only approved assets/materials; Supplier only sees RFQs they’re invited to.

## 2) Endpoint Scopes (by Milestone)

Prefix: `/api/*`

| Area (Milestone)    | Endpoints                                                               | Roles             | Notes                                                 |
|---------------------|-------------------------------------------------------------------------|-------------------|-------------------------------------------------------|
| Projects (M1)       | `GET/POST /projects`, `GET/PUT/DELETE /projects/:id`                      | Owner, Designer   | Client read selective (by invitation)                 |
| Assets (M1)         | `POST /projects/:id/assets/upload-url`, `GET /projects/:id/assets`, `GET /assets/:id` | Owner, Designer   | R2 signed PUT, short expiry; Client only approved assets |
| BOQ & P&L (M2)      | `POST /projects/:id/boq/calc`, `POST /projects/:id/boq/snapshot`, `GET /projects/:id/boq/snapshots` | Owner, Designer   | Client read of snapshots (summary)                    |
| Materials (M3)      | `GET/POST /materials`, `GET /materials/:id`                               | Owner, Designer   | Catalog management                                    |
| RFQ (M3)            | `POST /projects/:id/rfq`, `GET /projects/:id/rfq`, `POST /rfq/:id/send`     | Owner, Designer   |                                                       |
| Supplier Portal (M6)| `POST /rfq/:id/respond`, `GET /suppliers/:id/quotes`                      | Supplier          | Validates invitation + RFQ status                     |
| Automation (M4)     | `POST /events/trigger`, `GET /events/logs`                                | Owner             | Signed webhooks / Zero-Trust                          |
| Collab (M5)         | `POST /assets/:id/lock`, `POST /assets/:id/unlock`, `GET /assets/:id/revisions` | Owner, Designer   | Durable Object enforces locks                         |

## 3) Authentication & Session
- **End-users (Owner/Designer/Client/Supplier):** Firebase Auth (email/OAuth).
  - Required: email verified; tokens ≤ 60 min; refresh securely.
  - Custom claims: `role`, `org_id`. Example payload: `{"sub": "...", "email": "x@x", "role": "designer", "org_id": "ORG123"}`
- **Owner/Admin Console (`/owner/*`):** Cloudflare Zero-Trust (Access) with device posture/IP allowlists and short-lived sessions.
- **Server-to-server hooks:** Signed HMAC header (`X-Signature`), clock-skew ≤ 2 minutes.

## 4) Authorization (Row-Level Security in Workers)
Every mutating/read action must enforce:
```javascript
assert(user.role in allowedRoles);
assert(user.org_id === resource.org_id);
assert(project.member(user.id) || user.role === 'owner');
```
Never rely on client-side checks. All access control decisions happen in the Worker before DB/R2 calls.

## 5) Data Stores & Policies
- **PlanetScale via Hyperdrive (DB):**
  - Use parameterized SQL only; no string concatenation.
  - Restrict queries by `org_id`; index `(org_id, project_id)`.
  - Migration approvals: Owner-only (prod), via PR + review.
- **R2 (files):**
  - Uploads only via presigned PUT (≤ 15 min expiry; strict `content-type`).
  - Public reads: disabled; serve via authenticated Worker URLs or limited signed GET.
- **KV (flags/settings):**
  - Non-sensitive flags only. Secrets go to Workers secrets, not KV.
- **Durable Objects (collab rooms):**
  - Validate membership per request; store minimal state.
  - Lock TTL (e.g., 60s renew); auto-release on disconnect.
- **Queues (automation):**
  - Idempotency keys; poison message DLQ; redact PII in payloads.

## 6) Rate Limiting & WAF
- WAF on `/api/*` with baseline rules (bots, SQLi, XSS).
- Per-role rate limits, e.g.:
  - Public (unauth) → 30 req/min/IP
  - Authenticated → 300 req/min/user
  - Supplier submit → 10 req/min/supplier
- Burst handling via `Retry-After`.

## 7) Secrets & Config
- Store secrets in Cloudflare Workers secrets (`wrangler secret put`).
- Rotate DB credentials via Hyperdrive/PlanetScale dashboards (no GitHub storage).
- Separate dev/staging/prod resources (DBs, R2 buckets, KV namespaces, Queues).

## 8) Audit Logging
Log every mutation with immutable events:
`events(id, org_id, project_id, actor_id, role, action, target, hash, created_at)`
- Include request hash (method+path+body digest).
- Store no plaintext secrets/PII; only references.
- Retention: 365 days (staging 30 days).

## 9) Supplier Access Flow (M6)
1. Invite supplier → create limited-scope record.
2. Verify email + token.
3. Access only assigned RFQs and own quotes.
4. All uploads via presigned PUT; virus scan hook (if enabled).
5. Quotes immutable after deadline unless reopened by Owner.

## 10) Zero-Trust Example (Owner Console)
Cloudflare Access policy (illustrative):
- Include emails: `this4arun@gmail.com`
- Device posture: managed device OR WebAuthn.
- Country allowlist: AE (optional).
- Session TTL: 8 hours; idle: 30 minutes.

## 11) Security Testing Checklist (per PR)
- Unit tests cover allowed vs. denied paths (role, org mismatch).
- Endpoint rejects unauthenticated/expired tokens.
- R2 presigned URL requires correct `content-type`.
- DB queries constrained by `org_id`.
- No secrets in logs.
- Rate limits configured for new endpoints.
- Audit event emitted.

## 12) Incident Response (quick playbook)
1. **Contain:** flip feature flag in KV or kill-switch env var; revoke presigned URL issuance.
2. **Block:** tighten WAF rule/rate limit; rotate Worker secret(s).
3. **Scope:** query events by time/org; export suspicious range.
4. **Recover:** rotate DB creds (Hyperdrive), invalidate caches, redeploy.
5. **Report:** summarize impact (who/what/when), actions taken, preventive tasks.

---

**Owner contact:** this4arun@gmail.com / 0506485536
**Last updated:** 2025-09-27 07:09:46