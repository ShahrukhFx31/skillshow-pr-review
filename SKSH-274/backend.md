# Backend PR Review — skillshow (`SKSH-274`)

**Repo:** skillshow  
**Branch:** `SKSH-274-1`  
**Base:** `main...HEAD` @ `b7ce9cb0`  
**Re-reviewed:** 2026-06-09  
**Scope:** Server-side CSV table export (`POST /v1/table-export`) — Critical / High only  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Files changed (vs `main`):** 16 — `table-export` controller/service/routes/validation, `table-export.utils`, `table-export-format.utils`, `table-export-columns.constants`, `csv-export.utils`, `edit-request.service` (`listForExport`), `BaseController`, types, tests

**Findings:** 1 (0 Critical, 1 High) — **1 Open**, **2 Fixed**, **2 Accepted**

### Protected modules

No changes to `list-query.validation.ts`, `list-query-aggregation.utils.ts`, `list-row-repository.utils.ts`, `mongo-change-stream.service.ts`, `change-stream.utils.ts`, or audit-log frozen modules. Export reuses existing list services and aggregation paths for data fetch.

---

## GitHub comments (Open findings)

### 1. `src/base/BaseController.ts` line 46

**High (Global consistency / Contract):** `badRequest` signature changed from `(res, message, isDisplayMessage?)` to `(res, message, data?, isDisplayMessage?)` without migrating existing callers. `event.controller.ts` lines 31 and 120 still call `this.badRequest(res, message, true)` — `true` is now serialized as response `data` instead of `isDisplayMessage`. Fix callers to `badRequest(res, message, undefined, true)` or add an overload so the third boolean arg remains `isDisplayMessage` when not an object.

---

## GitHub comments (Accepted — intentional, no change)

### 2. `src/services/table-export.service.ts` line 37

**Accepted (2026-06-09):** `listExportQuery` without `search`/`status`/`role` filters is **intentional** — **Export All** exports the **full table dataset**, not the on-screen filtered subset. Page-level **Export X** covers filtered/page/selection export; server export is the full dump.

### 3. `src/validation/table-export.validation.ts` line 4

**Accepted (2026-06-09):** Body schema validating `{ table }` only is **intentional** — no filter fields on the export API by design.

---

## GitHub comments (Fixed — no action)

### 4. `src/services/table-export.service.ts` line 43 — row limit ✅

`assertWithinExportRowLimit` (lines 43–66) rejects exports when `total > TABLE_EXPORT_MAX_ROWS` (100k) with structured `data` (`TABLE_EXPORT_TOO_MANY_ROWS`). Frontend parses this in `tableExportService.ts`. **Fixed.**

### 5. `src/utils/table-export.utils.ts` line 30 — column parity ✅

Export columns and formatters mirror admin UI `csvValue` headers and labels. **Fixed.**

---

---
Export ignores active list filters (full-table export by design)

Risk Level: HIGH  
File Path: src/services/table-export.service.ts  
Lines: 37-41, 104-155  
File Path: src/validation/table-export.validation.ts  
Lines: 4-11  
File Path: src/services/edit-request.service.ts  
Lines: 459-460

Description:
**Contract:** `TableExportService` uses a static `listExportQuery`; `tableExportBodySchema` accepts `{ table }` only. Export does not apply UI list filters (`search`, status tabs, role, etc.).

**Accepted (2026-06-09):** Confirmed **intentional**. **Export All** = full table export; filtered/page export remains the separate **Export X** client path. `listForExport` with `buildListFilter({}, "user", userId)` follows the same rule for edit requests.

Optional follow-up (out of scope): row-limit copy (`TABLE_EXPORT_TOO_MANY_ROWS_MESSAGE`) says "narrow your filters" though the API has no filter params — consider rewording to reference the 100k cap only.

**PR comment (`table-export.service.ts` line 37):**  
**High:** `listExportQuery` has no search/status/role filters — Export All dumps the full table.  
**Accepted:** Intentional — full-table export by design.

**PR comment (`table-export.validation.ts` line 4):**  
**High:** Body schema only validates `table` — no filter fields can be passed from the client.  
**Accepted:** Intentional — no filter fields on export API by design.

**Status:** Accepted

---

---
`BaseController.badRequest` signature breaks existing callers

Risk Level: HIGH  
File Path: src/base/BaseController.ts  
Lines: 46-52  
File Path: src/controllers/event.controller.ts  
Lines: 31, 120

Description:
**Global consistency:** This PR adds an optional `data` third parameter to `badRequest`, shifting `isDisplayMessage` to the fourth position. `event.controller.ts` still passes `true` as the third argument (`badRequest(res, outcome.message, true)`), which now becomes response `data: true` in the JSON error body instead of controlling `isDisplayMessage`.

Impact:
- Event create/update validation errors return a spurious `data: true` field — API contract regression for unrelated endpoints touched only via the shared base class.
- Pattern will repeat if other controllers adopt the old 3-arg boolean form.

Recommendation:
Migrate `event.controller.ts` call sites:

```ts
return this.badRequest(res, outcome.message, undefined, true);
```

Or add a `badRequest` overload / named options object so existing `(res, message, isDisplayMessage)` call sites remain valid.

**PR comment (`BaseController.ts` line 46):**  
**High:** Adding `data` as the 3rd `badRequest` arg breaks `event.controller.ts` lines 31/120 — `true` is now emitted as response `data`. Please migrate callers or use an overload.

**PR comment (`event.controller.ts` line 31):**  
**High:** `badRequest(res, outcome.message, true)` — third arg is now `data`, not `isDisplayMessage`. Use `badRequest(res, outcome.message, undefined, true)`.

**Status:** Open

---

---
Silent truncation at row cap (re-review)

Risk Level: HIGH  
File Path: src/services/table-export.service.ts  
Lines: 43-66, 163

**Re-review:** ✅ **Fixed** — `assertWithinExportRowLimit` before CSV write; frontend handles `TABLE_EXPORT_TOO_MANY_ROWS`.

**Status:** ✅ Fixed

---

---
CSV column config diverges from UI (re-review)

Risk Level: HIGH  
File Path: src/utils/table-export.utils.ts  
Lines: 30-221

**Re-review:** ✅ **Fixed** — server export columns match UI `csvValue` shape.

**Status:** ✅ Fixed

---

## Positive notes

- Clean layering: route → controller → service → existing domain list services → `writeCsvFile`.
- RBAC in `assertCanExport` mirrors list access patterns.
- Clear product split: **Export X** (filtered/page/selection, client) vs **Export All** (full table, server).
- Row-limit guard with structured error payload; frontend parses `TABLE_EXPORT_TOO_MANY_ROWS`.
- Video-library export resolves event/athlete labels via `getLookups()` context.
- Temp file cleanup after `res.download`; UTF-8 BOM for Excel compatibility.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Export ignores active list filters (full-table by design) | HIGH | Accepted | src/services/table-export.service.ts | 37-41, 104-155 |
| 2 | `BaseController.badRequest` breaks callers | HIGH | Open | src/base/BaseController.ts | 46-52 |
| 3 | Silent truncation at row cap | HIGH | ✅ Fixed | src/services/table-export.service.ts | 43-66, 163 |
| 4 | CSV column config diverges from UI | HIGH | ✅ Fixed | src/utils/table-export.utils.ts | 30-221 |

**Merge readiness:** **Not merge-ready** — 1 open High finding (`BaseController.badRequest` regression on `event.controller.ts` lines 31, 120).
