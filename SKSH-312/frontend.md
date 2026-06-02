# Frontend PR Review — skillshow-admin-ui (`SKSH-312`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-312`  
**Base:** `main...HEAD`  
**Scope:** Seq-based ID migration in management flows (Critical/High/Medium)  
**Findings:** 1 (0 High, 1 Medium)

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

**PR comment (`use-app-user-actions.tsx` line 34):**  
**Medium:** Actions now navigate with `String(record.seq)` only. Please guard null/invalid `seq` rows (or hide them server-side) so admin actions don’t route to invalid URLs.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Navigation assumes valid `seq` on every row | MEDIUM | Accepted | src/pages/management/app-users/dashboard/hooks/use-app-user-actions.tsx | 34-43 |

**Positive notes:** Seq migration is consistently applied across app/crew/skillshow actions, columns, and route utilities.

**Merge readiness:** No open Critical/High/Medium blockers (seq-backfill dependency satisfied).
