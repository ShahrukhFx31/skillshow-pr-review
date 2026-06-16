# SkillShow Backend — Pre-Release Security Audit

**Scope:** `skillshow/src` — routes, middleware, controllers, services, config, super-admin surfaces  
**Date:** 2026-06-15  
**Method:** File-by-file review of authentication, RBAC, authorization bypasses, IDOR, privilege escalation, and deployment risks.

---

## Executive Summary

The most severe issues are **commented-out RBAC middleware** on admin user mutations, **missing ownership checks** on distribution/analytics read endpoints, and **unscoped S3 upload keys** that allow cross-user object overwrite. Several **system-role protections are disabled in code** (commented guards), creating privilege-escalation paths for any principal with `user` or `role` update permissions — and, worse, for **any authenticated user** on user create/delete because route-level `authorize` is commented out.

Super-admin routes themselves are correctly gated (`super_admin` + `admin` for dashboard read; `super_admin` only for cache flush). The larger risk is **inconsistent authorization** elsewhere: many `/v1/*` routes rely only on JWT presence, with enforcement deferred to controllers that is incomplete on several hot paths.

---

## Findings

### 1. Commented RBAC on user creation — any authenticated user can create admin accounts

| Field | Value |
|-------|-------|
| **Severity** | **CRITICAL** |
| **Location** | `src/routes/user.routes.ts:43-48`, `src/controllers/user.controller.ts:60-117`, `src/validation/user.validation.ts:27-38` |
| **Title** | Admin user creation endpoint has authorization bypassed |

**Description**  
`POST /api/v1/users` is protected by global JWT auth but the `authorize({ permissions: ["user"], operation: "create" })` middleware is **commented out** (line 45). The controller creates users with arbitrary `roleIds` (including `admin` / `super_admin` ObjectIds) and `isActive: true` with no server-side role restriction. Any logged-in user (athlete, parent, coach, etc.) can call this endpoint.

**Impact**  
Full account takeover of the platform: attacker creates an active admin-equivalent user, logs in, and gains access to all admin-only surfaces (`/v1/app-users`, `/v1/import-tool`, `/v1/settings`, etc.).

**Fix**  
1. Uncomment and enforce `authorize({ permissions: ["user"], operation: "create" })` on the route.  
2. In `UserController.createUser`, validate `roleIds` against an allowlist (block `admin`, `super_admin`, `crew`, `editor` unless caller is admin).  
3. Add integration tests asserting non-admin JWT receives 403.

---

### 2. Commented RBAC on user deletion — any authenticated user can soft-delete users

| Field | Value |
|-------|-------|
| **Severity** | **CRITICAL** |
| **Location** | `src/routes/user.routes.ts:100-106`, `src/controllers/user.controller.ts:481-503` |
| **Title** | User soft-delete endpoint has authorization bypassed |

**Description**  
`DELETE /api/v1/users/:id` has `authorize({ permissions: ["user"], operation: "delete" })` **commented out** (line 103). Controller performs soft-delete with no ownership or role check.

**Impact**  
Any authenticated user can soft-delete arbitrary user accounts (including admins), causing denial of service and data integrity loss.

**Fix**  
1. Restore `authorize` middleware on the route.  
2. Optionally prevent deletion of system/service accounts and self-deletion without elevated role.  
3. Add audit logging for user deletions.

---

### 3. System role modification guard is commented out

| Field | Value |
|-------|-------|
| **Severity** | **CRITICAL** |
| **Location** | `src/controllers/role.controller.ts:275-278` |
| **Title** | System roles (`admin`, `super_admin`, etc.) can be renamed or demoted |

**Description**  
`updateRole` contains a guard that would block modification of `role.system` roles, but it is **commented out**:

```typescript
// if (role.system && (name || system === false)) {
//   return this.badRequest(res, "Cannot modify system role");
// }
```

A principal with `role` **update** permission can rename the `admin` role, set `system: false`, or change metadata on seeded system roles. `assignPermissions` (line 510+) has no system-role check either.

**Impact**  
Privilege model corruption: attacker with role-management access can weaken or repurpose system roles, grant wildcard permissions to arbitrary roles, or break RBAC for all users.

**Fix**  
1. Uncomment and extend the system-role guard to block **any** mutation of system roles (name, description, permissions, `system` flag).  
2. Apply the same guard in `assignPermissions`.  
3. Only allow permission changes on non-system roles, or require `super_admin` for system role maintenance.

---

### 4. Distribution status/insights IDOR — no ownership validation before proxying

| Field | Value |
|-------|-------|
| **Severity** | **HIGH** |
| **Location** | `src/controllers/distribution.controller.ts:165-183`, `190-211`, `257-300`, `src/routes/distribution.routes.ts:13-59` |
| **Title** | Distribution read/insight endpoints forward requests without verifying video/job ownership |

**Description**  
`submit`, `retryAll`, and `retryVendor` correctly call `athleteService` ownership assertions. However, `getStatus`, `getStatusByVideoId`, `getInsights`, and `fetchInsights` only require authentication, then **proxy directly** to the orchestrator with `jobId` or `videoId` — no check that the caller owns the video or job.

**Impact**  
Any authenticated user who knows or guesses a `videoId`/`jobId` can read another user's distribution status, vendor logs, and platform insights (titles, URLs, publish metadata).

**Fix**  
Before `forward()`, resolve the video/job owner and assert `viewerUserId` matches owner or has an accepted parent/coach/admin relation (reuse `athleteService` helpers used on submit/retry). Return 404 on mismatch to avoid enumeration.

---

### 5. Analytics insights IDOR for orphan videos

| Field | Value |
|-------|-------|
| **Severity** | **HIGH** |
| **Location** | `src/controllers/analytics.controller.ts:45-55`, `src/routes/analytics.routes.ts:16-21` |
| **Title** | Video insights query allows access when `video.user` is null/missing |

**Description**  
`getVideoInsights` builds a Mongo filter that matches videos where `user` equals the caller **OR** `user` is null/missing:

```typescript
$or: [
  { user: userId },
  { user: { $exists: false } },
  { user: null },
],
```

Any video record without an owner is readable by **every** authenticated user.

**Impact**  
Cross-user disclosure of vendor analytics, distribution job metadata, and insight time-series for orphaned or improperly created video documents.

**Fix**  
Remove the null/missing-user branch for non-admin callers. Restrict orphan access to admin/service roles only. Prefer explicit `admin` RBAC on this route.

---

### 6. S3 upload keys are not user-scoped — cross-user file overwrite risk

| Field | Value |
|-------|-------|
| **Severity** | **HIGH** |
| **Location** | `src/services/s3.service.ts:79-81`, `src/controllers/upload.controller.ts:16-33`, `src/controllers/video.controller.ts:31-57`, `src/constants/upload.constants.ts:6` |
| **Title** | Predictable global S3 keys (`uploads/{fileName}`) without per-user prefix |

**Description**  
`keyFromFileName()` produces `uploads/{sanitizedFileName}` with **no userId**. Any authenticated user can request a presigned PUT for `video.mp4` and overwrite another user's object. `POST /api/v1/videos` accepts an arbitrary `key` from the body with no validation that the key belongs to the caller's namespace.

**Impact**  
Data destruction, content spoofing, and potential linkage of attacker-controlled S3 objects to victim video records.

**Fix**  
1. Prefix all upload keys with `uploads/{userId}/` (or UUID per upload session).  
2. On video create, validate `key` starts with the caller's allowed prefix.  
3. Consider S3 bucket policies conditioning writes on key prefix via STS/session policy.

---

### 7. Role assignment without protected-role validation

| Field | Value |
|-------|-------|
| **Severity** | **HIGH** |
| **Location** | `src/controllers/user.controller.ts:350-359`, `src/routes/user.routes.ts:65-72` |
| **Title** | `POST /api/v1/users/:id/roles` allows assigning privileged roles |

**Description**  
`assignRoles` validates role IDs exist but does **not** block assignment of `admin`, `super_admin`, `crew`, or `editor` to arbitrary users. Route requires `user` **update** permission (enforced), but any principal with that permission (including `super_admin` seed permissions) can escalate any account.

**Impact**  
Horizontal/vertical privilege escalation: compromised admin-adjacent account or over-permissioned role can grant full admin to attacker-controlled user.

**Fix**  
1. Maintain a `PROTECTED_ROLE_NAMES` constant; only `admin`/`super_admin` may assign them.  
2. Log all role assignments with actor, target, and role names.  
3. Require re-authentication or MFA for privileged role grants (future hardening).

---

### 8. Global video stats cache flush by any authenticated user

| Field | Value |
|-------|-------|
| **Severity** | **MEDIUM** |
| **Location** | `src/controllers/video.controller.ts:394-408`, `src/routes/video.routes.ts:28-31` |
| **Title** | `DELETE /api/v1/videos/stats/cache` flushes all users' stats cache |

**Description**  
Comment says "Flush the video stats cache for the current user" but implementation calls `cacheService.flushByPrefix` on the entire `video-stats` namespace — deleting **all** users' cached stats keys, not just `userId`.

**Impact**  
Cache denial-of-service: any authenticated user can repeatedly flush shared Redis keys, increasing DB load for all users.

**Fix**  
Flush only `buildCacheKey({ namespace: "video-stats", parts: [userId] })` for the caller. Require admin RBAC if global flush is intentionally needed.

---

### 9. Socket.IO auth weaker than HTTP auth

| Field | Value |
|-------|-------|
| **Severity** | **MEDIUM** |
| **Location** | `src/middleware/socket.middleware.ts:11-30` vs `src/middleware/auth.middleware.ts:49-85` |
| **Title** | WebSocket connections skip `tokensValidAfter`, `isActive`, and `isDeleted` checks |

**Description**  
HTTP `authenticate` validates user active status and `tokensValidAfter` (post-password-reset invalidation). Socket middleware only verifies JWT signature — revoked/compromised tokens remain valid on WebSocket until JWT expiry.

**Impact**  
After password reset or account deactivation, old access tokens may still receive real-time notifications and join user rooms until token TTL expires.

**Fix**  
Reuse the same post-verify user lookup and `tokensValidAfter` logic from `auth.middleware.ts` in `registerSocketAuthMiddleware`.

---

### 10. JWT accepted via query string and logged by Morgan

| Field | Value |
|-------|-------|
| **Severity** | **MEDIUM** |
| **Location** | `src/middleware/auth.middleware.ts:20-21`, `src/app.ts:92-100` |
| **Title** | Bearer token leakage via URL query param and access logs |

**Description**  
`extractToken` accepts `?token=` and `headers.token`. Morgan logs full request URLs. Tokens in query strings can appear in access logs, browser history, Referer headers, and CDN/proxy logs.

**Impact**  
Session hijacking if tokens are passed via URL (intentional or via misconfigured clients).

**Fix**  
1. Remove query-string token fallback for production (or gate behind `NODE_ENV === 'development'`).  
2. Redact `token` query params in Morgan format.  
3. Document Authorization-header-only policy for clients.

---

### 11. Swagger UI exposed without authentication

| Field | Value |
|-------|-------|
| **Severity** | **MEDIUM** |
| **Location** | `src/config/swagger.ts:150-152`, `src/app.ts:158` |
| **Title** | Full API documentation publicly served at `/api-docs` |

**Description**  
Swagger is mounted unconditionally with no auth, environment gate, or IP restriction. It documents auth flows, admin endpoints, and request/response shapes.

**Impact**  
Attack surface reconnaissance: lowers cost of targeted attacks against admin and auth endpoints in production.

**Fix**  
Disable Swagger in production (`NODE_ENV === 'production'`) or protect with admin auth / network ACL / basic auth.

---

### 12. OAuth state stored in process memory (multi-instance breakage + weak binding)

| Field | Value |
|-------|-------|
| **Severity** | **MEDIUM** |
| **Location** | `src/utils/vendor/handlers/index.ts:23-83`, `src/app.ts:116-121` |
| **Title** | In-memory OAuth state store; unauthenticated `/v1/vendors/complete` |

**Description**  
OAuth `state` is stored in a per-process `Map` (max 10k entries). In horizontally scaled deployments, callback may hit a different instance than the one that issued `state`, breaking OAuth — or encouraging auth bypass workarounds. `POST /api/v1/vendors/complete` is intentionally unauthenticated; security depends entirely on one-time `consumeState(state)`.

**Impact**  
1. Production OAuth failures under load balancing.  
2. If state validation is weakened, attacker could bind victim OAuth codes to attacker `userId`.  
3. State store is not shared across instances (Redis recommended).

**Fix**  
1. Move OAuth state to Redis with TTL and single-use delete (like refresh tokens).  
2. Ensure `consumeState` is atomic across instances.  
3. Rate-limit `/v1/vendors/complete` per IP.

---

### 13. User search exposes PII without permission check

| Field | Value |
|-------|-------|
| **Severity** | **MEDIUM** |
| **Location** | `src/routes/user.routes.ts:22-26`, `src/repositories/user.repository.ts:242-244` |
| **Title** | `GET /api/v1/users/search` returns email for parent/coach/editor users |

**Description**  
Endpoint is auth-only (no RBAC). Returns `_id`, `username`, `firstName`, `lastName`, `displayName`, **`email`**, and `profileImageKey` for users matching a typeahead query, scoped by role (`parent`, `coach`, `editor`).

**Impact**  
Authenticated users can enumerate emails and identities of parents, coaches, and editors — privacy violation and phishing aid. Partially intentional for "Add Connection" but lacks rate limits and role-based gate.

**Fix**  
1. Require relation-specific permission or limit to users with an existing athlete context.  
2. Return masked email (e.g. `j***@domain.com`) for non-admin searchers.  
3. Add per-user rate limiting and minimum query length (≥3 chars).

---

### 14. Table export route uses no-op RBAC (`authorize({})`)

| Field | Value |
|-------|-------|
| **Severity** | **MEDIUM** |
| **Location** | `src/routes/table-export.routes.ts:9-14`, `src/services/table-export.service.ts:68-93` |
| **Title** | Export endpoint relies solely on service-layer checks |

**Description**  
`authorize({})` only verifies JWT presence. `assertCanExport` enforces admin for sensitive tables and `Events` read for events, but `edit-requests` case **returns without an authorization check** (line 92-93) — relying on `listForExport(userId)` scoping.

**Impact**  
Defense-in-depth gap: misconfiguration in `listForExport` would expose all edit requests. Route gives no hint of required permissions to future maintainers.

**Fix**  
1. Replace `authorize({})` with explicit per-table middleware or a single `authorize` mapping.  
2. Add explicit permission check for `edit-requests` export (admin or edit-request admin permission).  
3. Align route RBAC with `assertCanExport` logic.

---

### 15. Super-admin role does not inherit `admin` role on role-gated routes

| Field | Value |
|-------|-------|
| **Severity** | **MEDIUM** (design / consistency) |
| **Location** | `src/routes/app-user.routes.ts:19`, `src/routes/super-admin.routes.ts:18-22`, `src/mocks/role.seed.ts:169-199` |
| **Title** | `super_admin` cannot access `authorize({ roles: ["admin"] })` routes unless also assigned `admin` |

**Description**  
Admin surfaces (`app-users`, `crew-users`, `import-tool`, `video-library`, `settings`, etc.) check `roles: ["admin"]` exactly. `super_admin` is a separate role with `user`/`event`/`dashboard` permissions in seed data but is **not** included in admin route checks. Super-admin dashboard read intentionally allows both `super_admin` and `admin`.

**Impact**  
Operational confusion and potential over-permissioning: teams may assign **both** roles to super admins, or grant `super_admin` users broad `user` CRUD permissions while admin routes stay blocked — leading to ad-hoc permission workarounds (e.g. commenting out `authorize`).

**Fix**  
1. Define canonical hierarchy: `super_admin` implies `admin` in `RBACService.hasAnyRole` **or** add `super_admin` to all admin route `roles` arrays.  
2. Document intended access model in `super-admin.constants.ts`.  
3. Avoid duplicating role checks between permission-based and role-based routes.

---

### 16. `assignPermissions` bypasses Mongoose soft-delete hooks via raw collection

| Field | Value |
|-------|-------|
| **Severity** | **MEDIUM** |
| **Location** | `src/controllers/role.controller.ts:355-387`, `577` (similar pattern) |
| **Title** | Direct collection access restores soft-deleted role-permission rows |

**Description**  
`RolePermissionModel.collection.findOneAndUpdate` is used intentionally to bypass pre-find hooks that filter soft-deleted documents. This restores deleted role-permission bindings on upsert.

**Impact**  
If combined with finding #3 (system role edits), attacker can resurrect prior permission grants or bypass intended permission revocation history.

**Fix**  
1. Block permission mutations on system roles (see finding #3).  
2. Use explicit repository methods with audit trail instead of raw collection upserts.  
3. Log permission matrix changes with before/after snapshots.

---

### 17. Auth endpoints lack dedicated brute-force rate limits

| Field | Value |
|-------|-------|
| **Severity** | **MEDIUM** |
| **Location** | `src/app.ts:75-85`, `src/routes/auth.routes.ts`, `src/services/otp.service.ts` |
| **Title** | Only global rate limit (100 req / 5 min) protects login/OTP/register |

**Description**  
Account lockout exists for login (`MAX_LOGIN_ATTEMPTS`) and OTP has per-code attempt limits, but `/api/auth/login`, `/api/auth/otp`, `/api/auth/register`, and password-reset endpoints share the coarse global IP limiter. No per-email or per-route throttling.

**Impact**  
Credential stuffing and OTP brute-force across many IPs; registration spam; email bombing via resend-verification.

**Fix**  
Add stricter per-route limiters (e.g. 5 login attempts / 15 min per IP+email). Consider CAPTCHA on register and password reset after N failures.

---

### 18. Broad `endsWith` auth bypass in conditional auth

| Field | Value |
|-------|-------|
| **Severity** | **LOW** |
| **Location** | `src/app.ts:124-131` |
| **Title** | Any `/v1/.../check-username` path skips authentication |

**Description**  
`path.endsWith("/check-username")` and `endsWith("/username-suggestions")` bypass JWT for **any** matching suffix, not only `/v1/users/check-username`.

**Impact**  
Future routes accidentally named `.../check-username` would be unintentionally public.

**Fix**  
Use exact path matching: `path === "/v1/users/check-username"` (and suggestions), not `endsWith`.

---

### 19. `GET /api/v1/events/me` lacks Events permission check

| Field | Value |
|-------|-------|
| **Severity** | **LOW** |
| **Location** | `src/routes/event.routes.ts:46-50` |
| **Title** | Event upload picker list accessible to any authenticated user |

**Description**  
Unlike other event routes, `GET /me` applies `authenticate` but not `authorize({ permissions: ["Events"], operation: "read" })`.

**Impact**  
Any logged-in user can list active platform events (likely low sensitivity if events are semi-public).

**Fix**  
Add Events read permission if this data should be staff-only; otherwise document as intentional.

---

### 20. Upload `POST /record` has no request validation middleware

| Field | Value |
|-------|-------|
| **Severity** | **LOW** |
| **Location** | `src/routes/upload.routes.ts:47`, `src/controllers/upload.controller.ts:135-147` |
| **Title** | Unvalidated body on upload record endpoint |

**Description**  
`recordUpload` accepts arbitrary JSON body without Joi validation (unlike presigned endpoints). Returns a public S3 URL for any key.

**Impact**  
Minor: inconsistent validation; could be used to probe arbitrary keys if combined with other issues.

**Fix**  
Apply `presignedUrlBodySchema` or a dedicated `recordUploadBodySchema`; validate key against caller namespace (see finding #6).

---

### 21. Public client-error ingestion endpoint

| Field | Value |
|-------|-------|
| **Severity** | **LOW** |
| **Location** | `src/routes/client-error.routes.ts:31-36`, `src/routes/index.ts:52` |
| **Title** | Unauthenticated error reporting with DB persistence |

**Description**  
`POST /api/client-errors` is public, rate-limited (30/min/IP), and may persist payloads to MongoDB depending on `ClientErrorService` rules.

**Impact**  
Log injection / DB storage abuse if rate limits are insufficient at scale; PII in error payloads from clients.

**Fix**  
1. Strip/limit payload size and field allowlist.  
2. Consider requiring auth for persisted errors.  
3. Monitor collection growth.

---

### 22. Required secrets default to empty strings

| Field | Value |
|-------|-------|
| **Severity** | **LOW** (deployment) |
| **Location** | `src/config/app.ts:66-68`, `115-118` |
| **Title** | `DISTRIBUTION_SERVICE_TOKEN` and `VIDEO_URL_KEY_MASTER_SECRET` optional with `""` default |

**Description**  
Empty distribution token may weaken orchestrator auth if the downstream service accepts missing tokens. Empty video URL master secret may degrade encrypted play-url security.

**Impact**  
Misconfigured production deploy could run with weak inter-service auth or predictable crypto.

**Fix**  
Fail fast at startup in production if these secrets are empty. Document in deployment checklist.

---

### 23. Refresh token rotation implemented correctly (informational)

| Field | Value |
|-------|-------|
| **Severity** | **INFO** |
| **Location** | `src/services/auth.service.ts:686-711` |
| **Title** | Refresh token rotation on use — good practice |

**Description**  
`refreshToken` revokes the used refresh token before issuing a new pair. This mitigates refresh token theft replay.

**Impact** | Positive security control.  
**Fix** | None required; ensure refresh tokens are stored hashed if not already.

---

### 24. Super-admin cache flush correctly restricted (informational)

| Field | Value |
|-------|-------|
| **Severity** | **INFO** |
| **Location** | `src/routes/super-admin.routes.ts:31-35` |
| **Title** | Dashboard cache flush is `super_admin` only |

**Description**  
`DELETE /api/v1/super-admin/dashboard/cache` uses `superAdminOnlyAuth` while dashboard read allows `admin` OR `super_admin`. This is appropriate separation of duties.

**Impact** | Positive — admin cannot flush shared super-admin cache.  
**Fix** | None.

---

### 25. Public registration role allowlist (informational)

| Field | Value |
|-------|-------|
| **Severity** | **INFO** |
| **Location** | `src/validation/auth.validation.ts:126-128`, `src/services/auth.service.ts:247-265` |
| **Title** | Self-registration correctly limits role to athlete/parent/coach |

**Description**  
`POST /api/auth/register` validates `role` as one of `athlete`, `parent`, `coach` only. Cannot self-register as admin via public register.

**Impact** | Positive control on public signup path.  
**Fix** | None — but finding #1 bypasses this via authenticated admin create endpoint.

---

## Priority Remediation Order

| Priority | Finding # | Action |
|----------|-----------|--------|
| P0 — before release | 1, 2 | Restore `authorize` on user create/delete immediately |
| P0 — before release | 3, 7 | Re-enable system role guards; restrict role assignment |
| P1 — before release | 4, 5, 6 | Add ownership checks on distribution/analytics; scope S3 keys per user |
| P1 — soon after | 8, 9, 10, 11 | Cache flush scope; socket auth parity; token logging; Swagger gate |
| P2 — hardening | 12–22 | Rate limits, search privacy, export RBAC, super_admin model, config validation |

---

## Files Reviewed (representative)

- **Middleware:** `auth.middleware.ts`, `rbac.middleware.ts`, `socket.middleware.ts`, `validate.middleware.ts`, `athleteRelation.middleware.ts`
- **App/Config:** `app.ts`, `config/app.ts`, `config/swagger.ts`, `routes/index.ts`
- **Super-admin:** `super-admin.routes.ts`, `super-admin.controller.ts`, `super-admin.service.ts`, `super-admin.constants.ts`
- **RBAC surfaces:** `user.routes.ts`, `role.routes.ts`, `permission.routes.ts`, `role.controller.ts`, `rbac.service.ts`
- **High-risk domains:** `distribution.controller.ts`, `analytics.controller.ts`, `upload.controller.ts`, `video.controller.ts`, `auth.service.ts`, `vendor/handlers/index.ts`
- **Admin modules:** `app-user.routes.ts`, `import-tool.routes.ts`, `setting.routes.ts`, `table-export.service.ts`

---

## Testing Recommendations

1. **Automated RBAC matrix tests** — for each `/v1` route, assert 401 without token, 403 with wrong role, 200 with correct role.  
2. **IDOR tests** — distribution/analytics with user A's token accessing user B's `videoId`.  
3. **Regression tests** — uncommented `authorize` on user routes must pass only with `user` permission.  
4. **S3 key isolation test** — user A cannot presign PUT to user B's key prefix.  
5. **Pen test** — focus on findings #1–#6 before production traffic.

---

*This audit is point-in-time against the repository state on 2026-06-15. Re-run after fixes and before each major release.*
