# Backend PR Review — skillshow (`SKSH-274`)

**Repo:** skillshow  
**Branch:** `SKSH-274-1`  
**Base:** `main...HEAD` @ `23edacec`  
**Reviewed:** 2026-06-09  
**Scope:** Server-side CSV table export (`POST /v1/table-export`) — Critical / High only  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Files changed (vs `main`):** 12 — `table-export` controller/service/routes/validation, `list-export.utils`, `csv-export.utils`, `edit-request.service` (`listForExport`), constants/types, tests

**Findings:** 3 (0 Critical, 3 High) — **3 Open**

### Protected modules

No changes to `list-query.validation.ts`, `list-query-aggregation.utils.ts`, `list-row-repository.utils.ts`, `mongo-change-stream.service.ts`, `audit-log.utils.ts`, or `change-stream.utils.ts`. Export correctly reuses existing list services and aggregation paths for data fetch.

---

## GitHub comments (Open findings)

### 1. `src/services/table-export.service.ts` line 27

**High (Contract / behavioral):** `listExportQuery` is a fixed default (no `search`, `status`, `role`, `eventType`, etc.). `fetchRows` (lines 69–93) passes only that query to every list service. The HTTP body schema (`table-export.validation.ts` line 4) only accepts `{ table }`, so **Export All** returns the full dataset regardless of admin UI filters. Please extend the export contract to accept the same list-query fields per table and pass them through `fetchRows`, or document that export is intentionally unfiltered and rename the UI action.

### 2. `src/services/table-export.service.ts` line 108

**High (behavioral):** `buildExportFile` maps all fetched rows to CSV (lines 108–117) with no check against `pagination.total`. `TABLE_EXPORT_MAX_ROWS` is 50_000 (`table-export.constants.ts` line 15) but truncation is silent. Admins with >50k rows may receive an incomplete CSV with no indication. Compare total before export and return `400` with a clear message, or paginate/stream internally and surface truncation in the filename or a response header.

### 3. `src/utils/list-export.utils.ts` line 30

**High (DRY / Global consistency):** `TABLE_EXPORT_COLUMN_CONFIG` hand-rolls headers and `toRow` mappers (lines 30–237) that duplicate and diverge from frontend table `csvValue` column configs — e.g. events server export includes Slug/Listing Status/City/State while page export uses Location; partners server export omits Logo. Two export buttons on the same dashboard produce inconsistent files. Centralize column definitions or derive server mappers from the same schema the UI uses.

---

---
Export ignores active list filters (search, status, tabs)

Risk Level: HIGH  
File Path: src/services/table-export.service.ts  
Lines: 27-31, 69-101  
File Path: src/validation/table-export.validation.ts  
Lines: 4-11

Description:
**Contract / behavioral:** `TableExportService` fetches rows with a static `listExportQuery` (`DEFAULT_LIST_QUERY` + `pageSize: TABLE_EXPORT_MAX_ROWS`) and no user-supplied filters. `tableExportBodySchema` only validates `table`. Every list call in `fetchRows` therefore omits `search`, `status`, `partnerType`, `role`, `eventType`, `listingStatus`, video-library `format`, and edit-request name/status/date filters that the admin UI applies to the on-screen table.

`edit-request.service.listForExport` hard-codes `buildListFilter({}, "user", userId)`, so edit-request export ignores toolbar search and filters even though `buildListFilter` supports them.

Impact:
- Admin on **Active Partners** with a search term exports inactive and unfiltered partners — data leak relative to user expectation and regression from the prior SKSH-274 client-side export-all that respected filters.
- Same mismatch on events (active/inactive tab), app/crew/skillshow users (status/role), video library (status/format), and edit requests (name/status/date).

Recommendation:
Extend `TableExportRequestBody` and `tableExportBodySchema` with optional list-query fields per `TableExportKey` (reuse each entity's `*ListQuery` type / Joi schema subset). Merge validated filters into `fetchRows` instead of the static `listExportQuery`. For edit requests, pass the UI query into `listForExport` (or call `fetchList` with the same params as `list`).

```ts
// table-export.validation.ts — example shape
export const tableExportBodySchema = Joi.object({
  table: Joi.string().valid(...TABLE_EXPORT_KEYS).required(),
  filters: Joi.object().unknown(true).optional(), // or per-table discriminated union
});
```

**PR comment (`table-export.service.ts` line 27):**  
**High:** `listExportQuery` has no search/status/role filters and `fetchRows` always uses it — Export All dumps the full table. Please accept the same list-query params the UI sends, or rename the action if a full dump is intentional.

**PR comment (`table-export.validation.ts` line 4):**  
**High:** Body schema only validates `table` — no filter fields can be passed from the client. Extend schema alongside `fetchRows`.

**PR comment (`edit-request.service.ts` line 378):**  
**High:** `listForExport` calls `buildListFilter({}, …)` — edit-request export ignores toolbar search/status/date filters.

**Status:** Open

---

---
Silent truncation at 50k rows

Risk Level: HIGH  
File Path: src/services/table-export.service.ts  
Lines: 30, 108-117  
File Path: src/constants/table-export.constants.ts  
Lines: 15

Description:
`TABLE_EXPORT_MAX_ROWS` is passed as `pageSize` to list services and as `maxRows` to `listForExport`. When the collection has more than 50,000 matching documents, the CSV is silently truncated — no error, no header, no log warning beyond `rowCount` in info logs.

Impact:
- Compliance/reporting exports may be incomplete without admin awareness.
- Hard to diagnose when exported row count does not match dashboard totals.

Recommendation:
After fetch, compare `rows.length` to `pagination.total` (or run a count first). If `total > TABLE_EXPORT_MAX_ROWS`, throw `400` with a message like "Export limited to 50,000 rows; apply filters to narrow results." Alternatively implement chunked export or streaming for large tables.

**PR comment (`table-export.service.ts` line 108):**  
**High:** `buildExportFile` writes all fetched rows with no total check — exports silently truncate at 50k. Please fail fast or warn when `total > TABLE_EXPORT_MAX_ROWS`.

**PR comment (`table-export.constants.ts` line 15):**  
**High:** `TABLE_EXPORT_MAX_ROWS = 50_000` is not surfaced to the client when the dataset exceeds this cap.

**Status:** Open

---

---
Duplicate CSV column config diverges from UI exports

Risk Level: HIGH  
File Path: src/utils/list-export.utils.ts  
Lines: 30-237

Description:
**DRY / Global consistency:** `TABLE_EXPORT_COLUMN_CONFIG` hand-rolls headers and row mappers for all seven tables. The admin UI already defines export columns via `csvValue` on table column configs (`partners-columns.tsx`, `event-columns.tsx`, `app-users-columns.tsx`, etc.) and uses `export-to-csv` for the page-level **Export X** buttons. Server and client exports now produce different column sets and labels for the same entity (events: server has Slug/Listing Status/City/State; UI has Location; partners: server lacks Logo column; app users: different name field handling).

Impact:
- Admins get inconsistent CSVs depending on which export button they click.
- Column changes on the UI will not automatically flow to server export — guaranteed drift.

Recommendation:
Define one export column schema per table (headers + field accessors) in `src/constants/` or `src/types/` and consume it from `list-export.utils.ts`. Longer term, align with frontend via a shared contract or OpenAPI-generated types. At minimum, match the existing UI `csvValue` columns for each table in this PR.

**PR comment (`list-export.utils.ts` line 30):**  
**High:** `TABLE_EXPORT_COLUMN_CONFIG` duplicates UI `csvValue` columns but with different headers/fields (events, partners, users). Please align with frontend column defs or share one schema.

**Status:** Open

---

## Positive notes

- Clean layering: route → controller → service → existing domain list services → `writeCsvFile`.
- RBAC in `assertCanExport` mirrors list access patterns (admin role for user/partner tables; `Events` read permission for events; user-scoped edit requests).
- Joi validation on `table`; `authorize({})` + service-level permission checks.
- Temp file cleanup after `res.download`; UTF-8 BOM for Excel compatibility.
- Unit tests for CSV escaping and body schema validation.
- `edit-request.service.listForExport` correctly bypasses HTTP `MAX_PAGE_SIZE` for internal export fetch.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Export ignores active list filters | HIGH | Open | src/services/table-export.service.ts | 27-31, 69-101 |
| 2 | Silent truncation at 50k rows | HIGH | Open | src/services/table-export.service.ts | 30, 108-117 |
| 3 | Duplicate CSV column config diverges from UI | HIGH | Open | src/utils/list-export.utils.ts | 30-237 |

**Merge readiness:** **Not merge-ready** — 3 open High findings (filter contract, truncation visibility, column parity).
