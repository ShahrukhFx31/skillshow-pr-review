# Frontend PR Review — skillshow-admin-ui (`SKSH-312`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-312`  
**Base:** `main...HEAD`  
**Scope:** Seq-based ID migration in management flows (Critical/High/Medium)  
**Findings:** 1 (0 High, 1 Medium) — 0 Open, 1 Accepted  
**Re-reviewed:** 2026-06-02 (commits through `634159a7` / `60dbb86c`, merges from `main`)

> **Scope note:** Pagination omitted per request.

---

---
Navigation assumes every app-user row has a valid `seq`

Risk Level: MEDIUM  
File Path: src/pages/management/app-users/dashboard/hooks/use-app-user-actions.tsx  
Lines: 34-43  
File Path: src/pages/management/app-users/onboarding/utils.ts  
Lines: 169-174  
File Path: src/pages/management/app-users/dashboard/components/app-users-columns.tsx  
Lines: 44-54

Description:
The UI now routes actions via `String(record.seq)` and route matching via `String(detail.seq) === routeUserId`. If backend returns a row with null/invalid seq (possible until backfill), generated links become invalid (`/view/null` / `/view/0`) and detail hydration cannot match.

Impact:
- Broken view/edit/resend/delete actions for rows missing a sequence.
- Confusing navigation failures for admins during migration period.

Recommendation:
Guard actions and links when `seq == null` (show disabled action + `—` ID), or coordinate with backend to suppress null-seq rows until migration is complete.

**Status:** Accepted — relies on completed `User.seq` backfill; no new seq guards added in admin management UI.

**PR comment (`use-app-user-actions.tsx` line 34):**  
**Medium:** Actions now navigate with `String(record.seq)` only. Please guard null/invalid `seq` rows (or hide them server-side) so admin actions don’t route to invalid URLs.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Navigation assumes valid `seq` on every row | MEDIUM | Accepted | src/pages/management/app-users/dashboard/hooks/use-app-user-actions.tsx | 34-43 |

### Re-review notes (2026-06-02)

| Change | Verdict |
|--------|---------|
| Seq migration in app/crew/skillshow dashboards, columns, route utils | Unchanged pattern; consistent with backend |
| `634159a7` athlete onboarding redirect (`auth-redirect`, `protected-route`, `accountService`) | Parent/coach app onboarding — **not** admin `app-users` management; no change to prior finding |
| `60dbb86c` activity status labels (`activity/utils.ts`, columns) | Improvement; no new Medium+ issues |
| `video.constants.ts` diff vs `main` | Line-ending/format only; no functional delta |
| `onboarding/index.tsx` vs `main` | No diff — activity route still loads full detail when not in Add mode (pre-existing; out of prior SKSH-312 scope) |

**Positive notes:** Seq migration applied consistently; linked-user table links use `row.seq` like dashboard columns.

**Merge readiness:** No open Critical/High/Medium blockers for this PR scope.
