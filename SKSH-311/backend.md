# Backend PR Review — skillshow (`SKSH-311`)

**Repo:** skillshow  
**Branch:** `SKSH-311`  
**Base:** `main...HEAD`  
**Re-verified:** 2026-06-02 (full review) — @ `fc85863`  
**Scope reviewed:** All **30 files** in `git diff main...HEAD` (app-user + video-library server-side list/bulk; shared list-query/sort-map refactor across crew/partner/skillshow; tests).  
**Findings:** 2 prior (✅ Fixed) + **1 Open High** (app-user bulk)

**Aligned with:** [frontend.md](./frontend.md)

### Review coverage (full pass)

| File group | Reviewed |
|------------|----------|
| `app-user.*` (controller, service, repository, routes, validation, types, constants, tests) | ✅ Full diff |
| `video-library.*` (controller, service, repository, routes, validation, types, constants, tests) | ✅ Full diff |
| `crew-user` / `partner` / `skillshow-user` constants + repository + validation | ✅ Full diff |
| `list-query-aggregation.utils.ts` | ✅ Full diff |
| Prior blockers (directory, `validatedQuery`) | ✅ Re-checked on branch |

---

---
`/partners/directory` returns paginated payload (breaks quick-connect consumers)

Risk Level: CRITICAL  
File Path: src/routes/partner.routes.ts  
Lines: 20-24

**Re-review:** ✅ **Fixed** — `PartnerController.directory` calls `listPartnersDirectory()` and returns a flat array (no list query schema on `/directory`).

**Status:** ✅ Fixed

---

---
List controllers merge raw `req.query` when `validatedQuery` is absent

Risk Level: HIGH  
File Path: src/controllers/crew-user.controller.ts  
Lines: 31-34

**Re-review:** ✅ **Fixed** — `AppUserController.list` and `VideoLibraryController.list` use `validatedQuery` + `DEFAULT_LIST_QUERY` only.

**Status:** ✅ Fixed

---

---
App-user bulk status update regressed to full `patchAppUser` per row

Risk Level: HIGH  
File Path: src/services/app-user.service.ts  
Lines: 304-316

Description:
`bulkPatchAppUsers` no longer uses the batched path (`findListRowsBySeqs` + `updateManyActiveStatusByIds` + `touchModificationByUserIds`). It loops up to `BULK_UPDATE_MAX_ITEMS` (50) and calls `patchAppUser(id, { status })` for each id.

Each `patchAppUser` loads the user, saves, runs `syncRoleProfile`, and re-runs the full list aggregate (`findListRowBySeq`). That is far heavier than a status-only bulk update and was not required for the pagination/list work.

Impact:
- Bulk activate/deactivate of many app users can time out or spike DB load (50× full patch + profile sync vs one batch update).
- Mid-loop failures leave earlier rows updated and later rows unchanged (no transaction).

Recommendation:
For status-only bulk patches, restore the batched update path (or a dedicated `bulkUpdateStatusBySeqs` repository method). Reserve `patchAppUser` for single-row PATCH:

```ts
// Resolve seqs → mongo ids once, then:
await userRepository.updateManyActiveStatusByIds(mongoUserIds, status === "active");
await appUserRepository.touchModificationByUserIds(mongoUserIds, audit);
// return list rows for ids (single aggregate with $in, not N× patchAppUser)
```

**PR comment (line 314):** **High:** Bulk status update now calls full `patchAppUser` per selected user (including `syncRoleProfile`) instead of the previous batched `updateMany` path — please restore a lightweight bulk status update for up to 50 rows.

**Status:** Open

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | `/partners/directory` returns paginated payload | CRITICAL | ✅ Fixed | src/routes/partner.routes.ts | 20-24 |
| 2 | List controllers merge raw `req.query` fallback | HIGH | ✅ Fixed | src/controllers/crew-user.controller.ts | 31-34 |
| 3 | App-user bulk status update regressed to N× `patchAppUser` | HIGH | Open | src/services/app-user.service.ts | 304-316 |

## Positive notes

- App-user list: `listAppUserQuerySchema`, paginated `listRows` via `runListQueryAggregate`, RBAC pipeline unchanged.
- Video library list: `videoLibraryListQuerySchema`, `listRowsPaginated` with sort/search maps in `video-library.constants.ts`, `validatedQuery` on `GET /`.
- Sort/search field maps centralized in `*-constants.ts` and wired through Joi `sortByValues`.
- `buildUserDocListMatch` now requires explicit `searchFields` per entity (no hidden default in utils).
- Partner directory and list controllers follow established `validatedQuery` pattern on paginated routes.
- Existing `PATCH /bulk` for video library (up to 200 ids) unchanged and appropriate for non–status-only bulk UI paths.

**Merge readiness:** Blocked on High #3 (app-user bulk patch). See [frontend.md](./frontend.md) High #5 (video library bulk status client path).
