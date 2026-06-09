# Frontend PR Review — skillshow-admin-ui (`SKSH-274`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-274-1`  
**Base:** `main...HEAD` @ `5460457c`  
**Reviewed:** 2026-06-09  
**Scope:** Server-driven **Export All** via `TableExportButton` + `POST /v1/table-export` (Critical / High / Medium)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Files changed (vs `main`):** 14 — `tableExportService`, `TableExportButton`, `use-table-export`, `postBlob` API client, `download-blob`, constants, wiring on 7 list dashboards

**Findings:** 3 (0 Critical, 2 High, 1 Medium) — **3 Open**

> **Note:** Prior review (2026-06-03, branch `SKSH-274`) covered client-side paginated export-all. This branch replaces that approach with server-side CSV download. Findings below apply to `SKSH-274-1`.

### Protected modules

No changes to `pagination-bar.tsx`, `use-pagination.ts`, `use-server-table-controls.ts`, `table-sort.ts`, `antd.adapter.tsx`, `destructive-action-confirm-modal.tsx`, or `AuditLogTable.tsx`. List pages continue to use `useServerTableControls` for table data; export is a separate action — correct separation, but filter state is not forwarded (see finding #1).

---

## GitHub comments (Open findings)

### 1. `src/api/services/tableExportService.ts` line 43

**High (Contract):** `exportTable` POSTs only `{ table }` (line 43). `TableExportButton` (`table-export-button.tsx` line 20) calls `runExport(table)` with no filter props. Dashboard call sites (e.g. `events/dashboard/index.tsx` line 137) pass only `table`, while `listEvents` in the same file (lines 60–67) sends `debouncedSearch`, `eventTypeFilter`, and `status`. **Export All** ignores active filters. Pass the same list params the `queryFn` uses once the backend accepts them.

### 2. `src/pages/events/dashboard/index.tsx` line 137

**High (DRY / UX):** Line 129–136 **Export Event** uses client `export-to-csv` (current page/selection); line 137 **Export All** hits the server with a different column set (see backend `list-export.utils.ts`). Same pattern on `partners/dashboard/index.tsx` line 95, `management/app-users/dashboard/index.tsx` line 94, and sibling dashboards. Align server export columns with page `csvValue` configs or consolidate to one export path.

### 3. `src/components/table-export-button.tsx` line 17

**Medium (KISS / UX):** `disabled` only reflects `isExporting` (line 17) — no prop for filtered `total === 0`. Parent pages gate the sibling **Export X** button on `hasNoEvents` but always render `TableExportButton` enabled, so an empty filtered table can still trigger a full server export.

---

---
Export All does not send active table filters to API

Risk Level: HIGH  
File Path: src/api/services/tableExportService.ts  
Lines: 40-44  
File Path: src/components/table-export-button.tsx  
Lines: 7-26  
File Path: src/pages/events/dashboard/index.tsx (representative)  
Lines: 43-78, 137

Description:
**Contract:** `exportTable` posts `{ table }` only. `TableExportButton` accepts no filter/search/sort props and `useTableExport` has no access to `useServerTableControls` state. Every dashboard wired in this PR (events, partners, video library, app/crew/skillshow users, edit requests) applies filters to `queryFn` / `listEditRequests` but not to export.

Example — events page passes `debouncedSearch`, `eventTypeFilter`, and `status` (active/inactive tab) to `listEvents`, but `TableExportButton` sends only `PAGE_TABLE_EXPORT_KEY.events`.

Impact:
- **Export All** downloads data outside the user's current view (wrong rows, wrong status tab, ignored search).
- Regression from prior SKSH-274 behavior that fetched all pages **with current filters**.
- Cross-stack mismatch with backend export service (also unfiltered).

Recommendation:
Extend `TableExportButton` (or a page-level handler) to accept export params:

```tsx
// events/dashboard/index.tsx
<TableExportButton
  table={PAGE_TABLE_EXPORT_KEY.events}
  filters={{
    search: debouncedSearch || undefined,
    eventType: eventTypeFilter ?? undefined,
    status,
    sortBy,
    sortOrder,
  }}
/>
```

Update `exportTable` / `TableExportRequest` to include `filters` in the POST body. Keep `queryKey`-style parity: every input that affects the list should affect export.

**PR comment (`tableExportService.ts` line 43):**  
**High:** POST body is only `{ table }` — search, status tabs, and filters are ignored. Please forward the same params as the list `queryFn`.

**PR comment (`table-export-button.tsx` line 20):**  
**High:** `onClick` calls `runExport(table)` with no filter/search/sort args from the parent page.

**PR comment (`events/dashboard/index.tsx` line 137):**  
**High:** `TableExportButton` receives only `table`; `listEvents` above (lines 60–67) passes `debouncedSearch`, `eventTypeFilter`, `status` — export/list params are out of sync.

**Status:** Open

---

---
Dual export buttons produce inconsistent CSVs (page vs server)

Risk Level: HIGH  
File Path: src/pages/events/dashboard/index.tsx  
Lines: 129-137  
File Path: src/pages/partners/dashboard/index.tsx  
Lines: 87-95  
File Path: src/pages/management/app-users/dashboard/index.tsx  
Lines: 86-94

Description:
**DRY / Global consistency:** Each dashboard keeps the existing **Export X** button (current page rows or selection → client `export-to-csv` using column `csvValue` defs) and adds **Export All** (server blob download). Server CSV columns differ from UI column configs (see backend `list-export.utils.ts` vs `event-columns.tsx`, `partners-columns.tsx`, etc.). Row scope also differs: page export = current page/selection; server export = up to 50k unfiltered rows.

Impact:
- Admins receive different file shapes from two buttons on the same toolbar.
- Maintenance burden: column changes must be updated in two places.

Recommendation:
Either (a) make server export the single **Export All** path that uses the same column schema as page export and respects filters, retiring redundant client full-export logic, or (b) remove page export when server export covers the use case. Short term: document the difference in button labels (e.g. "Export Page" vs "Export Full Database") until columns align.

**PR comment (`events/dashboard/index.tsx` line 137):**  
**High:** **Export Event** (line 135) and **Export All** (line 137) on the same toolbar produce different CSV columns and row scope. Please align server export with `csvValue` defs or consolidate.

**PR comment (`partners/dashboard/index.tsx` line 95):**  
**High:** Same dual-export mismatch — **Export Partners** (line 93) vs `TableExportButton` (line 95).

**PR comment (`management/app-users/dashboard/index.tsx` line 94):**  
**High:** Same dual-export mismatch — **Export Users** (line 92) vs `TableExportButton` (line 94).

**Status:** Open

---

---
Export All enabled when filtered list is empty

Risk Level: MEDIUM  
File Path: src/components/table-export-button.tsx  
Lines: 11-26

Description:
Sibling **Export X** buttons are gated by `hasNoEvents` / `hasNoPartners` (no rows in system), but `TableExportButton` is always enabled when rendered. A user can apply filters that yield `total === 0` on the table yet still trigger a full unfiltered server export (thousands of rows).

Impact:
- Confusing UX: empty table but export still downloads unrelated data.
- Reinforces filter-mismatch issue when export is unfiltered.

Recommendation:
Pass `disabled={total === 0}` from parent pages (based on filtered `total`, not global meta). Once filters are forwarded, disable when filtered export would return zero rows.

**PR comment (`table-export-button.tsx` line 17):**  
**Medium:** `disabled` only checks `isExporting` — pass `disabled={total === 0}` from parents so Export All cannot run when the filtered list is empty.

**Status:** Open

---

## Positive notes

- Shared `TableExportButton` + `useTableExport` + `TABLE_EXPORT_KEYS` / `PAGE_TABLE_EXPORT_KEY` — good DRY wiring across seven dashboards.
- `apiClient.postBlob` correctly bypasses JSON response interceptor for successful blob responses; `resolveTableExportErrorMessage` parses blob error bodies for 403/4xx.
- `downloadBlob` + `resolveContentDispositionFileName` reuse pattern suitable for future file downloads.
- `suppressSuccessToast: true` on export POST avoids spurious success toasts.
- Edit-request toolbar integration is minimal and consistent with other pages.
- Protected table-control modules untouched; list pages still use `useServerTableControls` + `applyServerSort` + hidden pagination + `PaginationBar`.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Export All does not send active table filters | HIGH | Open | src/api/services/tableExportService.ts | 40-44 |
| 2 | Dual export buttons produce inconsistent CSVs | HIGH | Open | src/pages/events/dashboard/index.tsx | 129-137 |
| 3 | Export All enabled when filtered list is empty | MEDIUM | Open | src/components/table-export-button.tsx | 11-26 |

**Merge readiness:** **Not merge-ready** — 2 open High and 1 open Medium finding; coordinate with backend filter/column fixes.
