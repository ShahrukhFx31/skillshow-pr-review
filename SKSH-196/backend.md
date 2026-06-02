# Backend PR Review — skillshow (`SKSH-196`)

**Repo:** skillshow (main API)  
**Base:** `main...HEAD`  
**Branch:** `SKSH-196`  
**Re-reviewed:** 2026-05-26 (after developer fixes)  
**Scope:** Import tool (validate/import CSV), auth email for coach/parent import, validation reuse  
**Findings:** 5 (0 Critical, 0 Open, 5 Fixed)

> **Scope note:** Pagination / deferred-validation findings (500-row split, job status API, background validation, per-row deferred saves) are omitted per review request.

---

## Partner duplicate check scans entire partner collection

**Risk Level:** HIGH  
**Status:** ✅ Fixed  
**File Path:** `src/repositories/import-tool.repository.ts`  
**Lines:** 19-30

**Description (original):**  
`findExistingOrganizationNames` aggregated all non-deleted partners before filtering by CSV names.

**Re-verification:**  
Now uses `PartnerModel.find` with `organizationName: { $in: names.map(...) }` and `escapeRegexSource` for case-insensitive exact match. Query scope is driven by CSV chunk values, not catalog size.

**PR comment:** Resolved on current branch.

---

## N+1 role lookups during row import

**Risk Level:** HIGH  
**Status:** ✅ Fixed  
**File Path:** `src/services/import-tool/import-tool-row-importer.service.ts`  
**Lines:** 165, 271, 492-497

**Description (original):**  
`RoleModel.findOne` / per-row role queries on coach/parent/skillshow user imports.

**Re-verification:**  
`loadImportRoleIdCache` calls `roleRepository.findActiveIdsByRoleNames` once per `persistRows` / `runDeferredImport` chunk; `importCoachParentRow` and `importSkillshowUserRow` use the cached `Map`.

**PR comment:** Resolved — roles preloaded per import batch.

---

## Entire CSV stored in one job document (`rowsByKey`)

**Risk Level:** HIGH  
**Status:** ✅ Fixed  
**File Path:** `src/models/import-tool-job.model.ts`  
**Also:** `src/models/import-tool-job-row.model.ts`, `src/services/import-tool/import-tool-job.store.ts`

**Description (original):**  
All row state lived in a single job document `rowsByKey` Mixed field.

**Re-verification:**  
Job schema holds metadata only (`import-tool-job.model.ts`). Rows persist via `ImportToolJobRow` + `importToolJobRowRepository`; `saveJob` splits metadata and row upserts.

**PR comment:** Resolved — row storage split from job metadata.

---

## No maximum row count on CSV parse

**Risk Level:** HIGH  
**Status:** ✅ Fixed  
**File Path:** `src/utils/csv.utils.ts`  
**Lines:** 55-80

**Also:** `src/constants/import-tool.constants.ts` (`IMPORT_TOOL_MAX_ROWS = 10_000`)

**Description (original):**  
`parseCsvBuffer` had no row limit.

**Re-verification:**  
`parseCsvBuffer(buffer, maxRows)` throws `CsvTooManyRowsError`; `validateCsvUpload` passes `IMPORT_TOOL_MAX_ROWS` and maps to `too_many_rows` validate error. Tests in `tests/utils/csv.utils.test.ts`.

**PR comment:** Resolved — 10k row cap at parse time.

---

## Synchronous import in one HTTP request

**Risk Level:** HIGH  
**Status:** ✅ Fixed  
**File Path:** `src/services/import-tool/import-tool-row-importer.service.ts`  
**Lines:** 50-73, 150-263

**Also:** `src/routes/import-tool.routes.ts` (`GET /import/:jobId/status`)

**Description (original):**  
Full job import ran sequentially inside one `POST /import` until completion.

**Re-verification:**  
`queueImportFromJob` returns **202** with `ImportToolImportAccepted`, runs `runDeferredImport` in background with chunked progress (`IMPORT_TOOL_IMPORT_CHUNK_SIZE`). Status via `getImportStatus`. Row-key imports ≤ `IMPORT_TOOL_SYNC_IMPORT_MAX_ROWS` (50) remain synchronous by design; larger row-key batches queue async.

**PR comment:** Resolved for `jobId` imports — async + status endpoint. Small row-key batches still sync intentionally.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Partner duplicate check full collection scan | HIGH | ✅ Fixed | `src/repositories/import-tool.repository.ts` | 19-30 |
| 2 | N+1 role lookups on import | HIGH | ✅ Fixed | `src/services/import-tool/import-tool-row-importer.service.ts` | 492-497 |
| 3 | Unbounded `rowsByKey` BSON size | HIGH | ✅ Fixed | `src/models/import-tool-job-row.model.ts` | — |
| 4 | No CSV row count cap | HIGH | ✅ Fixed | `src/utils/csv.utils.ts` | 55-80 |
| 5 | Sync import single HTTP request | HIGH | ✅ Fixed | `src/services/import-tool/import-tool-row-importer.service.ts` | 50-73 |

**New issues on re-review:** None (Critical/High within scope).

**Positive notes:** Clear layer split; admin RBAC; Joi reuse; coach/parent welcome email; S3 CSV backup; job TTL; async import with conflict handling (`importStatus` pending/in_progress); `import-tool-import.async.test.ts` covers queue flow.

**Skipped (per request):** Deferred-validation failure leaves `validationComplete` false if background validate throws (unchanged in `validateDeferredRows` catch); pagination/deferred-validation API gaps.
