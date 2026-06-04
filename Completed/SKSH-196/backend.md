# Backend PR Review — skillshow (`SKSH-196`)

**Repo:** skillshow (main API)  
**Base:** `main...HEAD`  
**Branch:** `SKSH-196`  
**Re-reviewed:** 2026-05-26 (final pass — all prior findings addressed)  
**Scope:** Import tool (validate/import CSV), auth email for coach/parent import, validation reuse  
**Findings:** 5 (0 Critical, 0 Open, 5 Fixed)  
**Review standard:** [backend-system-prompt.md](../prompts/backend-system-prompt.md) (Critical/High, Status column, re-verify)

> **Scope note:** Pagination / deferred-validation findings are omitted per review request.

---

## Partner duplicate check scans entire partner collection

**Risk Level:** HIGH  
**Status:** ✅ Fixed  
**File Path:** `src/repositories/import-tool.repository.ts`  
**Lines:** 19-30

**Re-verification:** Scoped `PartnerModel.find` with escaped regex `$in` on CSV names — unchanged and correct.

---

## N+1 role lookups during row import

**Risk Level:** HIGH  
**Status:** ✅ Fixed  
**File Path:** `src/services/import-tool/import-tool-row-importer.service.ts`  
**Lines:** 492-497

**Re-verification:** `loadImportRoleIdCache` + `roleRepository.findActiveIdsByRoleNames` per batch — unchanged and correct.

---

## Entire CSV stored in one job document (`rowsByKey`)

**Risk Level:** HIGH  
**Status:** ✅ Fixed  
**File Path:** `src/models/import-tool-job-row.model.ts`, `src/services/import-tool/import-tool-job.store.ts`

**Re-verification:** Job metadata and rows split across collections — unchanged and correct.

---

## No maximum row count on CSV parse

**Risk Level:** HIGH  
**Status:** ✅ Fixed  
**File Path:** `src/utils/csv.utils.ts`  
**Lines:** 55-80

**Re-verification:** `IMPORT_TOOL_MAX_ROWS` (10k) + `CsvTooManyRowsError` — unchanged and correct.

---

## Synchronous import in one HTTP request

**Risk Level:** HIGH  
**Status:** ✅ Fixed  
**File Path:** `src/services/import-tool/import-tool-row-importer.service.ts`  
**Lines:** 50-73

**Also:** `src/routes/import-tool.routes.ts` (`GET /import/:jobId/status`)

**Re-verification:** `queueImportFromJob` → **202** + background chunked import + status endpoint — unchanged and correct.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Partner duplicate check full collection scan | HIGH | ✅ Fixed | `src/repositories/import-tool.repository.ts` | 19-30 |
| 2 | N+1 role lookups on import | HIGH | ✅ Fixed | `src/services/import-tool/import-tool-row-importer.service.ts` | 492-497 |
| 3 | Unbounded `rowsByKey` BSON size | HIGH | ✅ Fixed | `src/models/import-tool-job-row.model.ts` | — |
| 4 | No CSV row count cap | HIGH | ✅ Fixed | `src/utils/csv.utils.ts` | 55-80 |
| 5 | Sync import single HTTP request | HIGH | ✅ Fixed | `src/services/import-tool/import-tool-row-importer.service.ts` | 50-73 |

**New issues on re-review:** None (Critical / High within scope).

**All tracked backend issues are resolved on the current branch.**

**Positive notes:** Layer split; admin RBAC; Joi reuse; async import + status; row storage split; partner/skillshow user validation enhancements on merged `main`.

**Skipped (per request):** Background `validateDeferredRows` catch still only logs on failure (no `validationError` flag on job) — marked **Accepted** under pagination exclusion, not a merge blocker.

### Supplemental prompt-only pass (import-tool PR)

| Check | Result |
|--------|--------|
| Joi `validate()` on routes | ✅ `body` + `params` on validate, import, status |
| Admin RBAC | ✅ `authorize({ roles: ["admin"] })` on all routes |
| Layer split | ✅ No Mongoose in controller |
| `validatedQuery` | N/A — no query endpoints |
| Soft-delete on duplicate lookups | ✅ `isDeleted: false` on event/partner/user queries |
| Regex escaping | ✅ `escapeRegexSource` on partner org names |
| Swallowed errors | ✅ Import failure sets `importStatus: "failed"`; deferred validate catch only logs (Accepted, see above) |

**Merge readiness:** No open Critical or High blockers. All summary rows are ✅ Fixed.
