# Backend PR Review — skillshow (`SKSH-312`)

**Repo:** skillshow  
**Branch:** `SKSH-312`  
**Base:** `main...HEAD`  
**Scope:** App/Crew/Skillshow user ID migration to `User.seq` (Critical & High only)  
**Findings:** 1 (0 Critical, 1 High) — 0 Open, 1 Accepted

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

**Status update:** Accepted — backfill completed by developer/team; keeping this as an operational release note rather than a code blocker.

**PR comment (`app-user.repository.ts` line 80):**  
**High:** List keys now derive from `user.seq`, but routes accept seq-only params. Please ensure `User.seq` backfill is complete (or exclude null seq rows) so visible users are actionable.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Users without `seq` break list keys and routes | HIGH | Accepted | src/repositories/app-user.repository.ts | 80-81, 229-240 |

**Previously reported, now fixed:** Linked-user N+1 was removed in this branch. `resolveLinkedUserKey` and per-link lookups were deleted; linked users now read `seq` directly from populated relation data in `loadLinkedUsers`.

**Positive notes:** Consistent seq migration across repositories/services/validation/tests; `createAppUser` now hard-fails if sequence assignment is missing.

**Merge readiness:** No open High blockers. `User.seq` backfill confirmed complete; residual risk accepted.
