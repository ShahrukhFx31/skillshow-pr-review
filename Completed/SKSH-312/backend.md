# Backend PR Review — skillshow (`SKSH-312`)

**Repo:** skillshow  
**Branch:** `SKSH-312`  
**Base:** `main...HEAD`  
**Scope:** App/Crew/Skillshow user ID migration to `User.seq` (Critical & High only)  
**Findings:** 1 (0 Critical, 1 High) — 0 Open, 1 Accepted  
**Re-reviewed:** 2026-06-02 (commits through `35b7fa0`, merges from `main`)

> **Scope note:** Pagination omitted per request.

---

---
Users without `seq` break list keys and seq-only routes

Risk Level: HIGH  
File Path: src/repositories/app-user.repository.ts  
Lines: 80-81, 229-240  
File Path: src/validation/user-key.validation.ts  
Lines: 4-10

Description:
List projection uses `key` from `User.seq` (`""` when null), while read/update/delete paths validate only numeric seq via `userSeqParamSchema` and resolve with `findListRowBySeq`. Rows missing `seq` can appear in list payloads but cannot be resolved by route handlers.

Impact:
- Legacy users missing sequence are visible but not actionable from app-user admin UI.
- Duplicate empty keys (`""`) can destabilize row identity in clients.

Recommendation:
Run a one-time `User.seq` backfill before release; until backfill is complete, filter null-`seq` users out of app-user list responses.

**Status:** Accepted — `User.seq` backfill completed; no server-side filter for null `seq` in list pipeline (residual operational note only).

**PR comment (`app-user.repository.ts` line 80):**  
**High:** List keys now derive from `user.seq`, but routes accept seq-only params. Please ensure `User.seq` backfill is complete (or exclude null seq rows) so visible users are actionable.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Users without `seq` break list keys and routes | HIGH | Accepted | src/repositories/app-user.repository.ts | 80-81, 229-240 |

### Re-review notes (2026-06-02)

| Change | Verdict |
|--------|---------|
| Linked-user N+1 removed — `loadLinkedUsers` uses populated `linked.seq` / `athleteUser.seq` (no per-relation `findById`) | **Fixed** (prior finding) |
| `onboarded` list column now `{ $eq: ["$isOnboarded", true] }` (`35b7fa0`) | Aligns admin list with real onboarding state |
| Login/profile onboarding for parent/coach — `isAppUserOnboarded`, `POST /profile/onboarding/complete` with RBAC | Looks correct; authorized, user-scoped |
| Seq migration across app/crew/skillshow repos + `parseUserSeqParam` | Consistent; `createAppUser` fails if sequence assignment missing |

**Positive notes:** Consistent seq migration across repositories/services/validation/tests; crew `editor` excluded from app-user RBAC list roles.

**Merge readiness:** No open Critical/High blockers. Seq backfill accepted; N+1 regression fixed on branch.
