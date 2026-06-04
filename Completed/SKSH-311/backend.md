# Backend PR Review — skillshow (`SKSH-311`)

**Repo:** skillshow  
**Branch:** `SKSH-311`  
**Base:** `main...HEAD`  
**Re-verified:** 2026-06-02 (full pr-verify) — @ `f1ee00d`  
**Scope reviewed:** **Full PR diff** — all **37 files** in `git diff main...HEAD` per backend system prompt.  
**Findings:** 3 prior + **0 new** Critical/High — all **✅ Fixed**

**Aligned with:** [frontend.md](./frontend.md)

### Full review coverage

| File group | Reviewed |
|------------|----------|
| `app-user.*` (controller, service, repository, routes, validation, types, constants, tests) | ✅ |
| `video-library.*` (controller, service, repository, routes, validation, types, constants, tests) | ✅ |
| `partner.*` (controller, service, repository, routes, validation, bulk + `getById`) | ✅ |
| `crew-user.*` / `skillshow-user.*` (service bulk refactor, repository list rows, validation) | ✅ |
| `list-query-aggregation.utils.ts`, `list-row-repository.utils.ts`, `user-seq-list-rows.utils.ts` | ✅ |
| Prior blockers (directory, `validatedQuery`) | ✅ Re-verified |

### pr-verify counts

| Metric | Count |
|--------|-------|
| New findings added | 0 |
| Prior Open → Fixed this pass | 0 (already fixed on branch) |
| Remaining Open | 0 |

---

---
`/partners/directory` returns paginated payload (breaks quick-connect consumers)

Risk Level: CRITICAL  
File Path: src/routes/partner.routes.ts  
Lines: 21-24

**Full re-review:** ✅ **Fixed** — `/directory` has no list query validation; `PartnerController.directory` calls `listPartnersDirectory()` and returns a flat array.

**Status:** ✅ Fixed

---

---
List controllers merge raw `req.query` when `validatedQuery` is absent

Risk Level: HIGH  
File Path: src/controllers/crew-user.controller.ts  
Lines: 31-34

**Full re-review:** ✅ **Fixed** — Paginated list controllers in this PR (`app-user`, `crew-user`, `partner`, `skillshow-user`, `video-library`) use `validatedQuery` + `DEFAULT_LIST_QUERY` only. Reporting-manager endpoints use `validatedQuery` for optional `search`.

**Status:** ✅ Fixed

---

---
App-user bulk status update regressed to full `patchAppUser` per row

Risk Level: HIGH  
File Path: src/services/app-user.service.ts  
Lines: 304-327

**Full re-review:** ✅ **Fixed** — Batched path restored: `findListRowsByUserIds` → `updateManyActiveStatusByIds` → `touchModificationByUserIds` → return rows with `{ ...row, status }`. No `patchAppUser` loop. Aligns with refactored `bulkPatchCrewUsers` / `bulkPatchSkillshowUsers` on this branch.

**Status:** ✅ Fixed

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | `/partners/directory` returns paginated payload | CRITICAL | ✅ Fixed | src/routes/partner.routes.ts | 21-24 |
| 2 | List controllers merge raw `req.query` fallback | HIGH | ✅ Fixed | src/controllers/crew-user.controller.ts | 31-34 |
| 3 | App-user bulk status update regressed to N× `patchAppUser` | HIGH | ✅ Fixed | src/services/app-user.service.ts | 304-327 |

## Positive notes (full diff)

- Shared list-row helpers (`list-row-repository.utils.ts`, `user-seq-list-rows.utils.ts`) used for batched bulk updates across app/crew/skillshow users and partners.
- Partner bulk (`bulkPatchPartners`) uses `updateManyByPartnerIds`; crew/skillshow bulk batch status and optional field updates with one refresh query.
- Paginated aggregates for app users and video library; sort/search maps in entity constants.

**Merge readiness:** ✅ Clear — archived 2026-06-02.
