# Backend PR Review — skillshow (`SKSH-312`)

**Repo:** skillshow  
**Branch:** `SKSH-312` → **`SKSH-178-1`** (stacked PR; compare with `git diff SKSH-178-1...HEAD`, not `main`)  
**Scope:** User `seq` migration for app / crew / skillshow admin APIs (replaces `appUserId`, `crewUserId`, etc.) — Critical & High only  
**Findings:** 2 (0 Critical, 2 High)

> **Scope note:** Same base as admin-ui (`SKSH-178-1`). Pagination omitted per review request. App-users APIs landed on `SKSH-178-1`; **this PR** only changes seq-based keys, validation, and repositories/services/tests in the diff vs `SKSH-178-1`.

---

---
N+1 `findById` when resolving linked user sequences on detail

Risk Level: HIGH  
File Path: src/services/app-user.service.ts  
Lines: 402-408, 424-435, 445-456

Description:
This PR replaces `resolveLinkedUserKey` (per-link list aggregation) with `resolveLinkedUserSeq`, which calls `authRepository.findById` once per accepted relation in `loadLinkedUsers`. Detail responses still scale with link count.

Impact:
- `GET /v1/app-users/:id` (seq param) latency grows linearly with linked users.
- Extra MongoDB load for coaches/parents with large rosters.

Recommendation:
Batch-load sequences for distinct linked user ids (e.g. `userRepository.findLeanSeqByIds`) and map in memory instead of per-row `findById`.

**PR comment (`app-user.service.ts` line 405):**  
**High:** `resolveLinkedUserSeq` does `findById` per linked user. Please batch-fetch `seq` for all linked user ids in one query so detail latency does not scale with relation count.

---

---
Users without `seq` break list keys and seq-only routes

Risk Level: HIGH  
File Path: src/repositories/app-user.repository.ts  
Lines: 76-77, 219-247  
File Path: src/validation/user-key.validation.ts  
Lines: 4-10

Description:
List projection now sets `key` from `User.seq` (`$toString` of empty when null). Routes and bulk ops use `userSeqParamSchema` / `findListRowBySeq`, which reject non-numeric ids. Users missing `seq` can appear in the list with `key: ""` but cannot be opened, updated, or bulk-selected reliably. `createAppUser` adds a post-create `seq` check (line 189) but does not backfill existing rows.

Impact:
- Legacy accounts unreachable from the admin UI after this migration.
- Duplicate empty `key` values in list responses.

Recommendation:
Run a one-time backfill assigning `User.seq` before release; optionally `$match` out null `seq` in `listPipeline` until backfill completes. Document in release checklist (same for crew/skillshow list pipelines on this branch).

**PR comment (`app-user.repository.ts` line 76):**  
**High:** List `key` now comes from `user.seq`, but params require a positive integer. Confirm prod backfill for users missing `seq` — otherwise they appear in the grid but all seq-based routes fail.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | N+1 `findById` for linked user `seq` on detail | HIGH | Open | src/services/app-user.service.ts | 402-408 |
| 2 | Users without `seq` break list keys and routes | HIGH | Open | src/repositories/app-user.repository.ts | 76-77, 219-247 |

**Positive notes:** Consistent move to `user.seq` across app/crew/skillshow services and repositories; shared `userSeqParamSchema` / `parseUserSeqParam`; `createAppUser` fails fast if `seq` is not assigned; bulk patch uses `findListRowsBySeqs`; tests updated for seq-based params.

**Skipped (per request):** Unpaginated `listRows()` / full-list fetch (pre-existing on `SKSH-178-1`).

**Merge readiness:** Not merge-ready — 2 open High findings (detail N+1, `seq` backfill).
