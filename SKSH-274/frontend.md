# Frontend PR Review — skillshow-admin-ui (`SKSH-274`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-274-1`  
**Base:** `main...HEAD` @ `935ecf8b`  
**Re-reviewed:** 2026-06-09  
**Scope:** Server-driven **Export All** via `TableExportButton` + `POST /v1/table-export` (Critical / High / Medium)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Files changed (vs `main`):** 15 — `tableExportService`, `TableExportButton`, `use-table-export`, `postBlob` API client, `download-blob`, constants/types, wiring on 7 list dashboards

**Findings:** 0 (0 Critical, 0 High, 0 Medium) — **1 Fixed**, **2 Accepted**

### Protected modules

No changes to frozen table-control, audit-log, or theme modules.

---

## GitHub comments (Accepted — intentional, no change)

### 1. `src/api/services/tableExportService.ts` line 103

**Accepted (2026-06-09):** POST body `{ table }` only is **intentional** — **Export All** triggers a **full-table** server export. Filtered/page/selection export stays on the sibling **Export X** button (`onExportReady` + client CSV). No filter props on `TableExportButton` by design.

### 2. `src/components/table-export-button.tsx` line 17

**Accepted (2026-06-09):** Not disabling **Export All** when filtered `total === 0` is **intentional** — full-table export should remain available even when the current filtered view is empty (the export is not scoped to the filter).

---

## GitHub comments (Fixed — no action)

### 3. Dual export CSV column mismatch ✅

Server export columns align with UI `csvValue` defs. **Fixed.**

---

---
Export All does not send active table filters to API

Risk Level: HIGH  
File Path: src/api/services/tableExportService.ts  
Lines: 103  
File Path: src/components/table-export-button.tsx  
Lines: 7-20  
File Path: src/pages/events/dashboard/index.tsx  
Lines: 60-67, 137

Description:
`exportTable` posts `{ table }` only. Dashboards pass list filters to `queryFn` but not to `TableExportButton`.

**Accepted (2026-06-09):** Confirmed **intentional** with backend — **Export All** = full dataset; **Export X** = filtered page/selection. Matches product split documented in backend review.

**PR comment (`tableExportService.ts` line 103):**  
**High:** POST body is only `{ table }`.  
**Accepted:** Intentional — full-table export by design.

**PR comment (`table-export-button.tsx` line 20):**  
**High:** `runExport(table)` with no filter args.  
**Accepted:** Intentional — filters not part of Export All contract.

**PR comment (`events/dashboard/index.tsx` line 137):**  
**High:** `TableExportButton` only receives `table`.  
**Accepted:** Intentional — list filters apply to on-screen data, not full export.

**Status:** Accepted

---

---
Export All enabled when filtered list is empty

Risk Level: MEDIUM  
File Path: src/components/table-export-button.tsx  
Lines: 11-17

Description:
`TableExportButton` is not disabled when filtered `total === 0`.

**Accepted (2026-06-09):** Intentional — full-table export must remain available when the filtered view is empty; disabling on `total === 0` would block valid full exports.

**PR comment (`table-export-button.tsx` line 17):**  
**Medium:** `disabled` only checks `isExporting`.  
**Accepted:** Intentional — Export All is not scoped to filtered row count.

**Status:** Accepted

---

---
Dual export buttons produce inconsistent CSVs (re-review)

Risk Level: HIGH  
File Path: src/pages/events/dashboard/index.tsx  
Lines: 129-137

**Re-review:** ✅ **Fixed** — column parity with server export. Row scope differs by design (**Export Event** = page/selection, **Export All** = full table — **Accepted**).

**Status:** ✅ Fixed

---

## Positive notes

- Shared `TableExportButton` + `useTableExport` + `PAGE_TABLE_EXPORT_KEY` across seven dashboards.
- Clear UX pairing: **Export X** (current view) + **Export All** (full server dump).
- Robust blob/JSON error handling for `TABLE_EXPORT_TOO_MANY_ROWS`.
- Protected table-control modules untouched.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Export All does not send active table filters | HIGH | Accepted | src/api/services/tableExportService.ts | 103 |
| 2 | Export All enabled when filtered list is empty | MEDIUM | Accepted | src/components/table-export-button.tsx | 17 |
| 3 | Dual export CSV column mismatch | HIGH | ✅ Fixed | src/pages/events/dashboard/index.tsx | 129-137 |

**Merge readiness:** **Blocked on backend** — frontend has no open findings; resolve `BaseController.badRequest` regression (`SKSH-274` backend finding #2) before merge.
