# Backend PR Review — skillshow (`SKSH-319`)

**Repo:** skillshow (main API) — `https://github.com/fx31labs-mvp/skillshow.git`  
**Branch:** `SKSH-319`  
**Base:** `main...HEAD` @ `255685c`  
**Initial review:** 2026-06-12  
**Re-reviewed:** 2026-06-16 (`2666c77` landed — upload-date read-only validation removed)  
**Scope:** Video Library CSV import via import-tool (`videoLibrary` entity type) — validation context, row resolution, bulk patch apply (Critical / High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [frontend.md](./frontend.md)

**Findings:** 1 (0 Critical, 1 High) — **0 Open**, **1 Accepted**

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
| `src/validation/video-library-import.validation.ts` | Joi row schema (`videoName` required) |
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
| Upload-date read-only validation for edited cells | ❌ Removed in `2666c77` — conflicts with UX requirement |
| TZ mismatch false-positive on unmodified exports | ✅ Avoided by omitting date check |
| Protected list/audit modules untouched | ✅ |

### Positive notes

- **Pipeline quality:** Import resolution and bulk persistence stay clean and well-factored.
- **Validation quality:** `videoName` required schema and tests remain intact.
- **Performance:** Batched row load + `bulkWrite` per import chunk.

### Developer response (upload-date validation)

**Team requirement:** Upload date is non-editable, and if a user edits it, they must see a validation error.  
**Current state (`2666c77`):** Upload-date validation is removed, so edited values are silently ignored at import.

---

## GitHub comments (Open findings)

No open findings.

---

## Findings

---
Upload-date edit is silently ignored (missing read-only validation)

Risk Level: HIGH  
**Status:** Accepted  
File Path: src/utils/video-library-import.utils.ts  
Lines: 86-97 (`validateReadOnlyImportCells`)

Description:
**Contract / UX regression.** In `2666c77`, `VideoLibraryImportRowMetadata.uploadDate` was removed and `validateReadOnlyImportCells` now only checks `Uploaded by`.  
Result: if a user edits `Upload date`, validation passes and row imports, but upload date is never persisted or updated.

Impact:
- Users can edit a read-only column without any validation feedback
- Imported data appears inconsistent because user-entered Upload date is ignored

Recommendation:
Accepted per team direction for this ticket. Behavior is intentional for now: Upload date remains non-persistent and edits are ignored. Follow-up can be tracked separately if product chooses to surface explicit non-editable feedback again.

**PR comment:** Not required (accepted).

---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Upload-date edit is silently ignored (missing read-only validation) | HIGH | Accepted | src/utils/video-library-import.utils.ts | 86-97 | — |

**Merge readiness:** **Merge-ready** — no open Critical/High blockers (1 High accepted).
