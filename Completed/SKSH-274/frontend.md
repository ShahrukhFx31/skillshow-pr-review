# Frontend PR Review — skillshow-admin-ui (`SKSH-274`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-274-1`  
**Base:** `main...HEAD` @ `b67c9899`  
**Re-reviewed:** 2026-06-09 (correct branch verified)  
**Scope:** Server-driven **Export All** via `TableExportButton` + `POST /v1/table-export`  
**Prompts:** `frontend-system-prompt.md`

**Files changed (vs `main`):** 15 — `tableExportService`, `TableExportButton`, `use-table-export`, `postBlob`, `download-blob`, constants/types, wiring on 7 dashboards

**Findings:** 0 Open — **1 Fixed**, **2 Accepted**

### Protected modules

No changes to frozen table-control, audit-log, or theme modules.

---

## GitHub comments (Accepted — intentional)

### 1. `src/api/services/tableExportService.ts` line 103

**Accepted (2026-06-09):** POST `{ table }` only — **Export All** = full-table server export; **Export X** = filtered/page/selection.

### 2. `src/components/table-export-button.tsx` line 17

**Accepted (2026-06-09):** Not disabling on filtered `total === 0` — intentional.

---

## GitHub comments (Fixed)

### 3. Dual export CSV column mismatch ✅

Server export columns align with UI `csvValue` defs (backend `table-export.utils.ts`).

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Export All does not send active table filters | HIGH | Accepted | src/api/services/tableExportService.ts | 103 |
| 2 | Export All enabled when filtered list is empty | MEDIUM | Accepted | src/components/table-export-button.tsx | 17 |
| 3 | Dual export CSV column mismatch | HIGH | ✅ Fixed | src/pages/events/dashboard/index.tsx | 129-137 |

**Merge readiness:** **Approve for merge** — no open Critical/High/Medium findings.
