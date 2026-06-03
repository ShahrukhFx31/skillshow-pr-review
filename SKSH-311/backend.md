# Backend PR Review — skillshow (`SKSH-311`)

**Repo:** skillshow  
**Branch:** `SKSH-311`  
**Base:** `main...HEAD`  
**Re-verified:** 2026-06-02 (second pass) — @ `5d04c5a`  
**Scope reviewed:** All **22 files** in `git diff main...HEAD` (app-user pagination + list constant refactor). Partners/crew/skillshow pagination from the earlier SKSH-311 merge is already on `main`; spot-checked on branch, not re-diffed file-by-file.  
**Findings:** 2 prior (✅ Fixed) + **1 new High**

**Aligned with:** [frontend.md](./frontend.md)

### Review coverage (second pass)

| File group | Reviewed |
|------------|----------|
| `app-user.*` (controller, service, repository, routes, validation, types, constants, tests) | ✅ Full diff |
| `crew-user` / `partner` / `skillshow-user` constants + repository + validation | ✅ Full diff (maps moved to constants; no logic regressions) |
| `list-query-aggregation.utils.ts` | ✅ Full diff |
| Prior blockers (directory, `validatedQuery`) | ✅ Re-checked on branch |

---

---
`/partners/directory` returns paginated payload (breaks quick-connect consumers)

Risk Level: CRITICAL  
File Path: src/routes/partner.routes.ts  
Lines: 20-24

**Re-review:** ✅ **Fixed**

**Status:** ✅ Fixed

---

---
List controllers merge raw `req.query` when `validatedQuery` is absent

Risk Level: HIGH  
File Path: src/controllers/crew-user.controller.ts  
Lines: 31-34

**Re-review:** ✅ **Fixed** — includes new `AppUserController.list`.

**Status:** ✅ Fixed

---

---
App-user bulk status update regressed to full `patchAppUser` per row

Risk Level: HIGH  
File Path: src/services/app-user.service.ts  
Lines: 304-316

Description:
`bulkPatchAppUsers` no longer uses the batched path (`findListRowsBySeqs` + `updateManyActiveStatusByIds` + `touchModificationByUserIds`). It now loops up to `BULK_UPDATE_MAX_ITEMS` (50) and calls `patchAppUser(id, { status })` for each id.

Each `patchAppUser` loads the user, saves, runs `syncRoleProfile`, and re-runs the full list aggregate (`findListRowBySeq`). That is far heavier than a status-only bulk update and was not required for the pagination/list work.

Impact:
- Bulk activate/deactivate of many app users can time out or spike DB load (50× full patch + profile sync vs one batch update).
- Mid-loop failures leave earlier rows updated and later rows unchanged (no transaction).

Recommendation:
For status-only bulk patches, restore the batched update path (or a dedicated `bulkUpdateStatusBySeqs` repository method). Reserve `patchAppUser` for single-row PATCH, matching the previous implementation:

```ts
// Resolve seqs → mongo ids once, then:
await userRepository.updateManyActiveStatusByIds(mongoUserIds, status === "active");
await appUserRepository.touchModificationByUserIds(mongoUserIds, audit);
// return list rows for ids (single aggregate with $in, not N× patchAppUser)
```

**PR comment (line 313):** **High:** Bulk status update now calls full `patchAppUser` per selected user (including `syncRoleProfile`) instead of the previous batched `updateMany` path — please restore a lightweight bulk status update for up to 50 rows.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | `/partners/directory` returns paginated payload | CRITICAL | ✅ Fixed | src/routes/partner.routes.ts | 20-24 |
| 2 | List controllers merge raw `req.query` fallback | HIGH | ✅ Fixed | src/controllers/crew-user.controller.ts | 31-34 |
| 3 | App-user bulk status update regressed to N× `patchAppUser` | HIGH | Open | src/services/app-user.service.ts | 304-316 |

## Positive notes

- App-user list: `listAppUserQuerySchema`, paginated `listRows`, RBAC pipeline unchanged.
- Sort/search maps centralized in `*-constants.ts` and wired through Joi `sortByValues`.
- `findListRowsBySeqs` removal is fine if bulk path is reimplemented without N× `patchAppUser`.
- Partner list sort map intentionally drops `partnerType` / `status` (aligned with frontend sorter removal).

**Merge readiness:** Blocked on High #3 (app-user bulk patch). Frontend table work looks fine pending backend fix.
