# Backend PR Review — skillshow (`SKSH-319`)

**Repo:** skillshow (main API) — `https://github.com/fx31labs-mvp/skillshow.git`  
**Branch:** `SKSH-319`  
**Base:** `main...HEAD` @ `d96579e`  
**Initial review:** 2026-06-12  
**Re-reviewed:** 2026-06-12 (`d96579e` — upload-date validation re-added; prior fix reverted); **2026-06-12** (no new commits — finding #1 still open)  
**Scope:** Video Library CSV import via import-tool (`videoLibrary` entity type) — validation context, row resolution, bulk patch apply (Critical / High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [frontend.md](./frontend.md)

**Findings:** 1 (0 Critical, 1 High) — **1 Open**

### Protected modules

| Module | Status |
|--------|--------|
| `list-query.validation.ts`, `list-query-aggregation.utils.ts`, audit-log stack, change-stream modules | **Not modified** ✅ |

### Files reviewed (14 changed)

| File | Change |
|------|--------|
| `src/constants/import-tool.constants.ts` | `videoLibrary` entity, CSV headers, duplicate checks |
| `src/constants/video-library-import.constants.ts` | Row validation messages |
| `src/repositories/video-library.repository.ts` | `findImportRowsByLibraryIds`, `bulkUpdateByLibrarySeq` |
| `src/services/import-tool.service.ts` | Video-library validation context; `templateHeaders` header detection |
| `src/services/import-tool/import-tool-row-validator.service.ts` | Post-schema resolution for `videoLibrary` |
| `src/services/import-tool/import-tool-row-importer.service.ts` | Bulk persist chunk for `videoLibrary` |
| `src/services/video-library.service.ts` | `bulkApplyImportPatches` |
| `src/types/import-tool.types.ts` | `ImportToolVideoLibraryValidationContext` |
| `src/types/video-library-import.types.ts` | Import payload / resolution types |
| `src/utils/import-tool.utils.ts` | Entity config, `loadVideoLibraryImportValidationContext` |
| `src/utils/video-library-import.utils.ts` | Row resolution, read-only checks, serialize/deserialize |
| `src/validation/video-library-import.validation.ts` | Joi row schema (`videoName` required @ `d96579e`) |
| `tests/services/import-tool.service.test.ts` | Entity config + schema tests |
| `tests/utils/video-library-import.utils.test.ts` | Resolution unit tests |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Reuses import-tool validate → persist pipeline | ✅ |
| `video-library-import.utils` centralizes resolution / patch mapping | ✅ DRY |
| `bulkUpdateByLibrarySeq` + `bulkApplyImportPatches` (one `bulkWrite` per chunk) | ✅ KISS |
| `templateHeaders` includes read-only columns in header presence check | ✅ |
| `TABLE_EXPORT_VIDEO_LIBRARY_ATHLETE_SEPARATOR` aligned with admin UI `"; "` | ✅ |
| Client-side export date vs import read-only check | ❌ TZ mismatch — canonical format needed (#1); omit check rejected for UX |
| `d96579e` restored upload-date read-only validation | ✅ Accepted — required so user edits show validation error |
| Protected list/audit modules untouched | ✅ |

### Positive notes

- **`d96579e` restore is correct (team):** Upload-date read-only validation must stay so edited cells show **"This column cannot be edited"**; omitting the check (`1a9542a`) silently ignores user edits.
- **`d96579e` improvements:** `videoName` Joi required + negative test; read-only Uploaded-by / Upload-date tests split.
- **Performance:** Batched row load + `bulkWrite` per import chunk.

### Developer response (upload-date validation)

**Accepted:** `1a9542a` was reverted intentionally — without upload-date validation, users who edit the column get no error and assume the value saved. Upload date must remain non-editable with visible validation. **Open work:** use the same canonical date string on export (frontend) and in `mapImportRowsToValidationData` so TZ does not false-fail unmodified exports.

---

## GitHub comments (Open findings)

### 1. `src/utils/video-library-import.utils.ts` line 102

> **High (Global consistency):** Keep upload-date read-only validation — users who edit the cell must see **"This column cannot be edited"** (omit check is not acceptable). Fix TZ false-positives by building `expected.uploadDate` with the **same canonical helper** as frontend export (e.g. UTC `MM/DD/YYYY`), not server-local `formatTableExportDate` alone while export uses browser-local `formatDate`.

---

## Findings

---
Upload date read-only validation must stay; canonical export/validation date formatting needed

Risk Level: HIGH  
**Status:** Open  
File Path: src/utils/video-library-import.utils.ts  
Lines: 39-42, 98-104 (`uploadDate` in `mapImportRowsToValidationData` / `validateReadOnlyImportCells`)

Description:
**Global consistency.** `validateReadOnlyImportCells` compares CSV `Upload date` to `formatTableExportDate(row.createdAt, "")` (server-local `MM/DD/YYYY`). Frontend export uses browser-local `formatDate(row.uploadedAt)`. Unmodified exports can false-fail when browser TZ ≠ server TZ.

**Developer response (accepted):** `1a9542a` omitted this check to avoid TZ false positives; **`d96579e` correctly restored it.** Without validation, users who edit Upload date see no error while import never persists that field — worse UX than TZ edge cases. Upload date must stay read-only with validation on edit.

Impact:
- TZ mismatch blocks unmodified export → import workflow
- Omitting validation confuses users who edit Upload date

Recommendation:
Add shared canonical formatter for import/export upload dates (UTC or fixed TZ). Use in `mapImportRowsToValidationData` (line 41) and frontend `export-video-library-csv.ts` (line 22). Restore/add cross-TZ test: unmodified export passes; deliberate edit still returns `VIDEO_LIBRARY_IMPORT_READ_ONLY_COLUMN_MESSAGE`.

**PR comment:** `src/utils/video-library-import.utils.ts` line **102** — see GitHub comments above.

---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Upload date read-only validation must stay; canonical export/validation date formatting needed | HIGH | Open | src/utils/video-library-import.utils.ts | 98-104 | 102 |

**Merge readiness:** **Not merge-ready** — 1 open High cross-stack blocker. Fix: shared canonical upload-date formatting on export + validation (keep read-only check). See [frontend.md](./frontend.md) #1.
