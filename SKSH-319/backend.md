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
| Client-side export date vs import read-only check | ❌ Regression — see #1 |
| `1a9542a` omitted upload-date check + TZ regression test | ✅ Fixed (then reverted in `d96579e`) |
| Protected list/audit modules untouched | ✅ |

### Positive notes

- **`1a9542a` showed correct fix:** Commit omitted upload-date tamper check with comment *"CSV uses display formatting (browser/server TZ) and cannot be compared reliably"* and test `does not reject Upload date when display string differs from server export TZ`.
- **`d96579e` improvements:** `videoName` Joi required + negative test; read-only Uploaded-by / Upload-date tests split.
- **Performance:** Batched row load + `bulkWrite` per import chunk.

---

## GitHub comments (Open findings)

### 1. `src/utils/video-library-import.utils.ts` line 102

**PR comment (line 102):** **High (Cross-stack / Regression):** `d96579e` re-added string equality on `Upload date` after `1a9542a` correctly removed it (with a TZ regression test). Server `formatTableExportDate(createdAt)` vs browser `formatDate(uploadedAt)` still diverge across timezones. Re-apply the `1a9542a` approach (skip upload-date tamper check; anchor on Video ID + Uploaded by) or canonicalize UTC on both sides.

---

## Findings

---
Upload date read-only check reintroduced after correct fix (timezone regression)

Risk Level: HIGH  
**Status:** Open  
File Path: src/utils/video-library-import.utils.ts  
Lines: 39-42, 98-104 (`formatDate` / upload-date check in `validateReadOnlyImportCells`)

Description:
**Global consistency / Regression.** `validateReadOnlyImportCells` compares CSV `Upload date` to `formatTableExportDate(row.createdAt, "")` (server-local `MM/DD/YYYY`).

Commit **`1a9542a`** removed this check and documented why: *"CSV uses display formatting (browser/server TZ) and cannot be compared reliably to `createdAt`"*, with test coverage for Pacific export vs UTC server mismatch.

Commit **`d96579e`** reverted that fix — re-added `uploadDate` to `VideoLibraryImportRowMetadata`, restored string comparison, and flipped the test to expect rejection instead of tolerance.

The primary admin export path still formats dates in the **browser** (`export-video-library-csv.ts` → `formatDate(row.uploadedAt)`). Unmodified exports can fail with **"This column cannot be edited"** when browser TZ ≠ server TZ.

Impact:
- Blocks the documented export → fill Event/Athlete → import workflow for admins in many timezones
- Reintroduces a bug the branch already fixed once

Recommendation:
**Prefer re-applying `1a9542a`:** validate `Uploaded by` only for read-only tamper detection; keep `Upload date` in CSV for display but do not reject on string mismatch.

Alternative: canonical UTC `YYYY-MM-DD` on both export and validation (coordinate with [frontend.md](./frontend.md)).

**PR comment (`video-library-import.utils.ts` line 102):** See GitHub comments above.

---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Upload date read-only check reintroduced after correct fix (timezone regression) | HIGH | Open | src/utils/video-library-import.utils.ts | 98-104 | 102 |

**Merge readiness:** **Not merge-ready** — 1 open High regression on the primary export → import workflow. Re-apply `1a9542a` upload-date omission or align canonical date formatting with [frontend.md](./frontend.md).
