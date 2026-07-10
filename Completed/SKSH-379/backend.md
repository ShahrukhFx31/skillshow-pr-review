# PR review — SKSH-379 (backend)

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow` |
| PR | [#232](https://github.com/SkillshowFx/skillshow/pull/232) |
| Branch | `SKSH-379` → `main` |
| Head | `cbeb770b17898f6bbf6d3fd46682331a70ccc7af` |
| Scope | Realtime RBAC session refresh: `GET /v1/auth/me`, `permissions:updated` socket emit, `tokensValidAfter` + refresh-token revocation on revocation; SkillShow branding tweaks |
| Prompts | `pr-review/prompts/backend-system-prompt.md`, `SECURITY-AUDIT-PRE-RELEASE.md` |

## GitHub comments

_No open findings._

## Findings

---
Session invalidation does not revoke refresh tokens

Risk Level: CRITICAL
File Path: src/repositories/auth.repository.ts
Lines: 393-414

Description:
**Security** — `invalidateSessionsForUserIds` only set `User.tokensValidAfter`. `AuthService.refreshToken` could still issue new tokens from valid refresh records.

Impact:
- Revoked users could stay logged in until refresh-token expiry.

Recommendation:
Revoke refresh tokens in the same invalidation path.

**Re-verify:** ✅ Fixed — `RefreshTokenModel.updateMany` revokes active refresh tokens; repository test added.
---

---
Session invalidation only on permission revocation, not CRUD downgrade

Risk Level: HIGH
File Path: src/controllers/role.controller.ts
Lines: 416-417, 650-651

Description:
**Security** — Sessions were invalidated only when permission IDs left the role, not when CRUD flags were tightened.

Recommendation:
Invalidate whenever role permissions mutate.

**Re-verify:** ✅ Fixed — invalidation runs after every permission upsert in `updateRole` and `assignPermissions`.
---

---
Menu notify skips parentId and name permission-tree fields

Risk Level: HIGH
File Path: src/controllers/permission.controller.ts
Lines: 369-385

Description:
`notifyPermissionsUpdatedForPermission` did not run for `parentId` or `name` updates.

Recommendation:
Include both fields in the notify trigger.

**Re-verify:** ✅ Fixed — `menuFields` includes `"name"` and `"parentId"`.
---

---
`deleteRole` skips session invalidation and socket notify

Risk Level: HIGH
File Path: src/controllers/role.controller.ts
Lines: 482-494

Description:
**Security** / **Global consistency** — Other RBAC revocation paths invalidated sessions and emitted `permissions:updated`, but `deleteRole` only soft-deleted the role.

Recommendation:
Resolve `findUserIdsWithRole(id)` before delete and mirror `deletePermission`.

**Re-verify:** ✅ Fixed — `findUserIdsWithRole`, `invalidateSessionsForUserIds`, and `emitPermissionsUpdated` added; controller test asserts all three.
---

## Positive notes

- `rbac-session-notify.utils.ts` centralizes user lookup, socket emit, and session invalidation (DRY).
- `GET /api/v1/auth/me` reuses `LoginResponseService.buildUserSessionProfile` — same shape as login.
- `invalidateSessionsForUserIds` test covers both `tokensValidAfter` and refresh-token revocation.
- `deleteRole` now matches update/assign/delete permission revocation behavior.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Session invalidation does not revoke refresh tokens | CRITICAL | ✅ Fixed | src/repositories/auth.repository.ts | 393-414 |
| 2 | Session invalidation only on permission revocation, not CRUD downgrade | HIGH | ✅ Fixed | src/controllers/role.controller.ts | 416-417, 650-651 |
| 3 | Menu notify skips parentId and name permission-tree fields | HIGH | ✅ Fixed | src/controllers/permission.controller.ts | 369-385 |
| 4 | `deleteRole` skips session invalidation and socket notify | HIGH | ✅ Fixed | src/controllers/role.controller.ts | 482-494 |

**Merge readiness:** No open Critical/High/Medium blockers on backend.
