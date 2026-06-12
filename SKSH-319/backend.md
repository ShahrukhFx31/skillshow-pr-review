# Backend PR Review — skillshow (`SKSH-319`)

**Repo:** skillshow (main API) — `https://github.com/fx31labs-mvp/skillshow.git`  
**Branch:** `SKSH-319`  
**Base:** `main...HEAD` @ `6a7bb8b`  
**Initial review:** 2026-06-12  
**Scope:** Video Library CSV import via import-tool (`videoLibrary` entity type) — validation context, row resolution, bulk patch apply (Critical / High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [frontend.md](./frontend.md)

**Findings:** 1 (0 Critical, 1 High) — **1 Open**

### Protected modules

| Module | Status |
|--------|--------|
| `list-query.validation.ts`, `list-query-aggregation.utils.ts`, audit-log stack, change-stream modules | **Not modified** ✅ |

No list endpoints, bulk row ops, audit logs, or change streams in this diff.

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
| `src/validation/video-library-import.validation.ts` | Joi row schema |
| `tests/services/import-tool.service.test.ts` | Entity config + schema smoke tests |
| `tests/utils/video-library-import.utils.test.ts` | Resolution unit tests (new) |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Reuses import-tool validate → persist pipeline | ✅ |
| `video-library-import.utils` centralizes resolution / patch mapping | ✅ DRY |
| `bulkUpdateByLibrarySeq` + `bulkApplyImportPatches` (one `bulkWrite` per chunk) | ✅ KISS |
| `templateHeaders` includes read-only columns in header presence check | ✅ Fixed vs `headerToField` filter |
| `TABLE_EXPORT_VIDEO_LIBRARY_ATHLETE_SEPARATOR` aligned with admin UI `"; "` | ✅ |
| Backend table export + import validation both use `formatTableExportDate` | ✅ (server-side export path) |
| Client-side Video Library export date vs import read-only check | ❌ Cross-stack — see #1 |
| `bulkApplyImportPatches` skips `eventSlugExists` / `athleteUsernamesExist` re-check | ✅ Accepted — matches import-tool validate-then-import pattern; documented in method comment |
| Protected list/audit modules untouched | ✅ |

### Positive notes

- **Update-only semantics:** Import resolves Event/Athlete labels to slugs/usernames, validates read-only metadata, and bulk-patches — appropriate for export → edit → re-import workflow.
- **Tests:** `video-library-import.utils.test.ts` covers resolution, read-only columns, unknown lookups, and serialization.
- **Performance:** Batched `findImportRowsByLibraryIds` + `bulkWrite` avoids per-row updates during import.
- **Header contract:** `IMPORT_TOOL_VIDEO_LIBRARY_HEADERS` matches frontend `VIDEO_LIBRARY_CSV_HEADERS`; import-tool tests assert alignment.

---

## GitHub comments (Open findings)

### 1. `src/utils/video-library-import.utils.ts` line 102

**PR comment (line 102):** **High (Cross-stack / Contract):** Read-only `Upload date` validation compares the CSV cell string to `formatTableExportDate(row.createdAt)` on the **server** timezone, but the primary admin export path formats `uploadedAt` in the **browser** (`formatDate` in `export-video-library-csv.ts`). Unmodified exports can fail near UTC midnight boundaries. Compare a canonical instant (e.g. UTC `YYYY-MM-DD` or ISO) on both sides, or drop string equality for dates.

---

## Findings

---
Upload date read-only check mismatches client-exported CSV dates

Risk Level: HIGH  
**Status:** Open  
File Path: skillshow/src/utils/video-library-import.utils.ts  
Lines: 39-42, 96-104

Description:
**Global consistency / Contract.** `mapImportRowsToValidationData` stores `uploadDate` as `formatTableExportDate(row.createdAt, "")` (server-local `MM/DD/YYYY`). `validateReadOnlyImportCells` rejects the row when the CSV `Upload date` cell does not equal that string exactly.

The documented workflow exports from Video Library in the admin UI via `exportVideoLibraryRowsAsCsv`, which writes `formatDate(row.uploadedAt)` using the **browser** timezone (`skillshow-admin-ui/src/pages/videoLibrary/dashboard/export-video-library-csv.ts:20`). The same ISO instant can format to different calendar dates when browser TZ ≠ server TZ (e.g. `2026-06-12T03:00:00.000Z` → `06/11/2026` in US Pacific vs `06/12/2026` on a UTC server).

Impact:
- Admins following the export → fill Event/Athlete → import flow can get false **"This column cannot be edited"** errors on `Upload date` without having changed the cell
- Blocks import of otherwise valid rows; workaround requires manually editing dates to match server formatting

Recommendation:
Compare canonical instants, not locale-dependent strings. Options (pick one and align frontend export):

1. **UTC date-only:** `dayjs.utc(value).format("YYYY-MM-DD")` in both `mapImportRowsToValidationData` and admin export.
2. **ISO equality:** store/compare `row.createdAt.toISOString()` (or date portion) instead of formatted display strings.
3. **Relax check:** validate `Uploaded by` only; treat `Upload date` as informational since `Video ID` already anchors row identity.

Add a unit test with a fixed UTC timestamp where local formatting diverges.

**PR comment (`video-library-import.utils.ts` line 102):** See GitHub comments above.

---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Upload date read-only check mismatches client-exported CSV dates | HIGH | Open | skillshow/src/utils/video-library-import.utils.ts | 39-42, 96-104 | 102 |

**Merge readiness:** **Not merge-ready** — 1 open High cross-stack blocker on the primary export → import workflow. Resolve date canonicalization with [frontend.md](./frontend.md) before merge.
