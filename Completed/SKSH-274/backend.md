# Backend PR Review — skillshow (`SKSH-274`)

**Repo:** skillshow  
**Branch:** `SKSH-274-1`  
**Base:** `main...HEAD` @ `30515fe1`  
**Re-reviewed:** 2026-06-09 (correct branch verified)  
**Scope:** Server-side CSV table export (`POST /v1/table-export`) — Critical / High only  
**Prompts:** `backend-system-prompt.md`

**Files changed (vs `main`):** 20 — `table-export` feature, `BaseController`, `event.controller` caller fix, export utils/constants/types, tests

**Findings:** 0 Open — **3 Fixed**, **2 Accepted**

### Protected modules

No changes to frozen list-query, aggregation, change-stream, or audit-log modules.

---

## GitHub comments (Accepted — intentional)

### 1. `src/services/table-export.service.ts` line 37

**Accepted (2026-06-09):** `listExportQuery` without filters is **intentional** — **Export All** = full table; **Export X** = filtered/page/selection.

### 2. `src/validation/table-export.validation.ts` line 4

**Accepted (2026-06-09):** `{ table }` only on export body is **intentional**.

---

## GitHub comments (Fixed)

### 3. `src/services/table-export.service.ts` line 43 — row limit ✅

`assertWithinExportRowLimit` (lines 43–66) returns `400` with `TABLE_EXPORT_TOO_MANY_ROWS` structured `data` (100k cap). Called at line 163 before CSV write.

### 4. `src/utils/table-export.utils.ts` line 30 — column parity ✅

Export columns/formatters align with admin UI `csvValue` defs (`table-export-format.utils.ts`, `table-export-columns.constants.ts`). Tests: `table-export.utils.test.ts`, `table-export-format.utils.test.ts`.

### 5. `src/controllers/event.controller.ts` line 35 — `badRequest` callers ✅

**Fixed:** `badRequest(res, message, undefined, true)` at lines **35**, **124**. Tests assert `payload.data` is `undefined` (`event.controller.test.ts`).

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Export ignores active list filters (full-table by design) | HIGH | Accepted | src/services/table-export.service.ts | 37-41, 104-155 |
| 2 | `BaseController.badRequest` breaks callers | HIGH | ✅ Fixed | src/controllers/event.controller.ts | 35, 124 |
| 3 | Silent truncation at row cap | HIGH | ✅ Fixed | src/services/table-export.service.ts | 43-66, 163 |
| 4 | CSV column config diverges from UI | HIGH | ✅ Fixed | src/utils/table-export.utils.ts | 30-221 |

**Merge readiness:** **Approve for merge** — no open Critical/High findings.
