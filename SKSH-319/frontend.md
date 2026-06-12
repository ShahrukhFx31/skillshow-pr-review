# Frontend PR Review — skillshow-admin-ui (`SKSH-319`)

**Repo:** skillshow-admin-ui — `https://github.com/fx31labs-mvp/skillshow-admin-ui.git`  
**Branch:** `SKSH-319`  
**Base:** `main...HEAD` @ `7a82e318`  
**Initial review:** 2026-06-12  
**Scope:** Video Library import-tool integration — entity type, CSV template/sample, workflow hints, shared column labels with export (Critical / High only)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [backend.md](./backend.md)

**Findings:** 1 (0 Critical, 1 High) — **1 Open**

### Protected modules

| Module | Status |
|--------|--------|
| `pagination-bar.tsx`, `use-pagination.ts`, `use-server-table-controls.ts`, `table-sort.ts`, `destructive-action-confirm-modal.tsx`, `antd.adapter.tsx`, audit-log components | **Not modified** ✅ |

Import-tool wizard only; no server-driven list or audit-log UI changes.

### Files reviewed (10 changed)

| File | Change |
|------|--------|
| `src/pages/import-tool/constants.ts` | `videoLibrary` entity type + labels |
| `src/pages/import-tool/dashboard/components/entity-selected-banner.tsx` | Configurable `hint` prop |
| `src/pages/import-tool/dashboard/components/select-entity-step.tsx` | Video-library workflow hint |
| `src/pages/import-tool/dashboard/components/upload-csv-step.tsx` | Entity-specific upload copy |
| `src/pages/import-tool/dashboard/constants/select-entity-step.constants.ts` | Sample row for `videoLibrary` |
| `src/pages/import-tool/dashboard/hooks/use-import-tool-wizard.tsx` | Pass `entityType` to upload step |
| `src/pages/import-tool/dashboard/types/components.ts` | `hint`, `entityType` props |
| `src/pages/import-tool/dashboard/utils/import-sample.utils.ts` | `getVideoLibraryCsvHeaders` case |
| `src/pages/videoLibrary/dashboard/constants.ts` | Shared CSV labels, headers, hints, sample row |
| `src/pages/videoLibrary/dashboard/export-video-library-csv.ts` | Use `VIDEO_LIBRARY_CSV_COLUMN_LABELS` |

### DRY / KISS / Global consistency scan

| Check | Verdict |
|-------|---------|
| `VIDEO_LIBRARY_CSV_COLUMN_LABELS` / `VIDEO_LIBRARY_CSV_HEADERS` single source for export + import template | ✅ DRY |
| Import-tool reuses video-library constants (hints, headers, sample) | ✅ |
| `ImportEntityContractChecks` satisfied (`SAMPLE_ROW_BY_ENTITY`, labels, options) | ✅ |
| `entityType`-specific copy via simple conditionals (not over-abstracted) | ✅ KISS |
| CSV column order matches backend `IMPORT_TOOL_VIDEO_LIBRARY_HEADERS` | ✅ |
| Athlete separator `"; "` matches backend `TABLE_EXPORT_VIDEO_LIBRARY_ATHLETE_SEPARATOR` | ✅ |
| Upload date export format vs backend read-only validation | ❌ Cross-stack — see #1 |
| Protected table/pagination modules untouched | ✅ |

### Positive notes

- **Reusability:** Extracting `VIDEO_LIBRARY_CSV_COLUMN_LABELS` into `constants.ts` and wiring export through it prevents header drift between Video Library export and import-tool templates.
- **UX:** Workflow hints (`VIDEO_LIBRARY_IMPORT_WORKFLOW_HINT`, `VIDEO_LIBRARY_IMPORT_UPLOAD_HINT`) clearly describe export → edit → import.
- **Type safety:** `entityType` added to `UploadCsvStepProps`; compile-time entity maps cover `videoLibrary`.
- **Help page:** `IMPORT_HELP_*` copy derives entity list from `IMPORT_ENTITY_TYPES` — `videoLibrary` appears automatically.

---

## GitHub comments (Open findings)

### 1. `src/pages/videoLibrary/dashboard/export-video-library-csv.ts` line 20

**PR comment (line 20):** **High (Global consistency):** `formatDate(row.uploadedAt)` uses the browser timezone, but backend import validation compares `Upload date` to `formatTableExportDate` on the server. Align both sides on a canonical format (e.g. UTC `YYYY-MM-DD`) so unmodified exports pass read-only validation — see backend finding in `video-library-import.utils.ts`.

---

## Findings

---
Video Library CSV export date format breaks import read-only validation

Risk Level: HIGH  
**Status:** Open  
File Path: skillshow-admin-ui/src/pages/videoLibrary/dashboard/export-video-library-csv.ts  
Lines: 20

Description:
**Global consistency / Cross-stack.** The primary export path (`exportVideoLibraryRowsAsCsv`, wired from the Video Library table "Export Videos" action) formats the `Upload date` column with `formatDate(row.uploadedAt)`, which uses the admin's **browser-local** timezone (`DEFAULT_DATE_FORMAT` = `MM/DD/YYYY`).

Backend import validation ([backend.md](./backend.md) #1) compares that cell to `formatTableExportDate(row.createdAt)` on the **API server** timezone. The same timestamp can produce different date strings across zones, causing false read-only errors on otherwise unedited exports.

Note: the separate `TableExportButton` (server-side table export) formats dates on the server and would match validation — but the import-tool hints and table UX steer users through the client-side export path.

Impact:
- False validation failures on `Upload date` for admins outside the server timezone (or near UTC midnight)
- Core workflow (export → add Event/Athlete → import) blocked until users manually fix date strings

Recommendation:
Match backend canonicalization. For example, add a shared UTC helper and use it here:

```typescript
import utc from "dayjs/plugin/utc";
dayjs.extend(utc);

// In videoLibraryCsvRowEntries:
[VIDEO_LIBRARY_CSV_COLUMN_LABELS.uploadDate, dayjs.utc(row.uploadedAt).format("YYYY-MM-DD")],
```

Coordinate the exact format with the backend fix in `mapImportRowsToValidationData` / `validateReadOnlyImportCells`.

**PR comment (`export-video-library-csv.ts` line 20):** See GitHub comments above.

---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Video Library CSV export date format breaks import read-only validation | HIGH | Open | skillshow-admin-ui/src/pages/videoLibrary/dashboard/export-video-library-csv.ts | 20 | 20 |

**Merge readiness:** **Not merge-ready** — 1 open High cross-stack blocker. Fix with [backend.md](./backend.md) #1 before merge.
