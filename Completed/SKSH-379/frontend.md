# PR review — SKSH-379 (frontend)

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow-admin-ui` |
| PR | [#332](https://github.com/SkillshowFx/skillshow-admin-ui/pull/332) |
| Branch | `SKSH-379` → `main` |
| Head | `d0f2f53c596538f0e15db79c5759e04d667eeb1b` |
| Scope | Client session profile refresh via `GET /v1/auth/me`, `permissions:updated` socket + reconnect/focus/boot/token-refresh hooks; SkillShow branding |
| Prompts | `pr-review/prompts/frontend-system-prompt.md` |
| Paired backend | `pr-review/SKSH-379/backend.md` ([skillshow#232](https://github.com/SkillshowFx/skillshow/pull/232)) |

## GitHub comments

_No open findings._

## Findings

---
Focus refresh runs even when socket is connected

Risk Level: MEDIUM
File Path: src/hooks/use-permissions-realtime.ts
Lines: 45-48

Description:
**KISS** — Hook docs say the window-focus path is a fallback “when the socket is disconnected,” but `onWindowFocus` always called `refresh()` (subject only to a 60s cooldown). Connected clients therefore hit `GET /v1/auth/me` + query invalidation on every tab focus after cooldown, even though `permissions:updated` already drives live updates.

Impact:
- Extra `/auth/me` load and full React Query churn under normal multi-tab use.
- Masks whether the socket path alone is sufficient before shipping.

Recommendation:
Gate focus refresh on disconnect, matching the comment.

**Re-verify:** ✅ Fixed — `if (socket.connected) return;` before `refreshWithCooldown()` on focus.
---

---
`permissionsRevision` is unused for router remount

Risk Level: MEDIUM
File Path: src/store/userStore.ts
Lines: 108-132

Description:
**KISS** — `usePermissionsRevision` was exported but never read; `usePermissionRoutes` already recomputes from `useUserPermission()` when `userInfo` updates.

Recommendation:
Remove the dead revision counter or wire it into router remount keys.

**Re-verify:** ✅ Fixed — field and export removed.
---

---
Unbounded `invalidateQueries()` on every permissions refresh

Risk Level: MEDIUM
File Path: src/hooks/use-permissions-realtime.ts
Lines: 24-27

Description:
Every permissions refresh called `queryClient.invalidateQueries()` with no predicate, marking the entire cache stale.

Recommendation:
Invalidate only permission-/auth-sensitive query-key prefixes.

**Re-verify:** ✅ Fixed — `invalidatePermissionSensitiveQueries()` with predicate allowlist.
---

---
Socket reconnect does not refresh session

Risk Level: MEDIUM
File Path: src/hooks/use-permissions-realtime.ts
Lines: 41-54

Description:
**DRY** / **Global consistency** — `use-edit-request-realtime.ts` recovers missed events on reconnect; permissions hook did not, leaving stale menu/caches after a same-tab reconnect.

Recommendation:
Register `socket.io.on("reconnect", refreshWithCooldown)` with the existing cooldown.

**Re-verify:** ✅ Fixed — reconnect handler added with shared `refreshWithCooldown`.
---

---
Permission-sensitive query allowlist has gaps

Risk Level: MEDIUM
File Path: src/utils/permission-realtime.utils.ts
Lines: 22-49

Description:
**KISS** — Scoped invalidation omitted RBAC-gated roots such as `events`, `videos`, `video-library`, `partners`, and `settings`.

Recommendation:
Extend `PERMISSION_SENSITIVE_QUERY_ROOTS` with shared `*_QUERY_KEY` constants.

**Re-verify:** ✅ Fixed — allowlist now includes `events`, video/my-videos/video-library, `partners`, and `settings` roots.
---

## Positive notes

- `AuthApi.Me = "/v1/auth/me"` aligns with backend `session-auth.routes` under `/api/v1/auth`.
- Boot refresh in `ProtectedRoute` after persist hydration addresses stale persisted permissions on reload.
- Token refresh calling `refreshUserSession` keeps profile in sync when access tokens rotate.
- Reconnect + focus fallbacks share one cooldown helper; query invalidation uses imported key constants.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Focus refresh runs even when socket is connected | MEDIUM | ✅ Fixed | src/hooks/use-permissions-realtime.ts | 45-48 |
| 2 | `permissionsRevision` is unused for router remount | MEDIUM | ✅ Fixed | src/store/userStore.ts | — |
| 3 | Unbounded `invalidateQueries()` on every permissions refresh | MEDIUM | ✅ Fixed | src/hooks/use-permissions-realtime.ts | 24-27 |
| 4 | Socket reconnect does not refresh session | MEDIUM | ✅ Fixed | src/hooks/use-permissions-realtime.ts | 41-54 |
| 5 | Permission-sensitive query allowlist has gaps | MEDIUM | ✅ Fixed | src/utils/permission-realtime.utils.ts | 22-49 |

**Merge readiness:** No open Critical/High/Medium blockers on frontend.
