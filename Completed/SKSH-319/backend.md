# Backend PR Review — skillshow (`SKSH-319`)

**Repo:** skillshow (main API) — `https://github.com/fx31labs-mvp/skillshow.git`  
**Branch:** `SKSH-319`  
**Base:** `main...HEAD` @ `2a68324`  
**Initial review:** 2026-06-12  
**Re-reviewed:** 2026-06-16 (`6e790c2` — all read-only CSV column checks removed)  
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
| `src/utils/video-library-import.utils.ts` | Row resolution, serialize/deserialize (no read-only checks) |
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
| Read-only CSV column enforcement (`Uploaded by`, `Upload date`) | ✅ Accepted — intentionally not validated (`2666c77`, `6e790c2`) |
| TZ false-positives on unmodified exports | ✅ Avoided by omitting date checks |
| `rowMetadataById` / `uploadedBy` still built but unused in resolution | ⚠️ Minor dead code (not a merge blocker) |
| Protected list/audit modules untouched | ✅ |

### Positive notes

- **Pipeline quality:** Import resolution and bulk persistence are clean and well-factored.
- **Validation quality:** `videoName` required schema + negative test; event/athlete resolution covered by unit tests.
- **Performance:** Batched row load + `bulkWrite` per import chunk.
- **Cross-TZ tests:** Explicit tests confirm `Upload date` and `Uploaded by` edits do not block import.

### Developer response (read-only columns)

**Accepted team direction:** Read-only display columns (`Uploaded by`, `Upload date`) are not persisted on import. Enforcing string equality caused TZ false positives; omitting validation avoids confusing silent failures on edited display fields. Users edit **Video Name**, **Event**, and **Athlete** only.

---

## GitHub comments (Open findings)

No open findings.

---

## Findings

---
Read-only CSV display columns not validated on import (intentional)

Risk Level: HIGH  
**Status:** Accepted  
File Path: src/utils/video-library-import.utils.ts  
Lines: 84-96 (`resolveVideoLibraryImportRow` — no `validateReadOnlyImportCells`)

Description:
**Contract / UX (accepted).** Commits `2666c77` and `6e790c2` removed read-only validation for `Upload date` and `Uploaded by`. Edits to those columns pass validation and are ignored at persistence (only `videoName`, `eventId`, `athleteIds` are applied).

Impact:
- Users can edit display-only columns without validation feedback
- Avoids TZ mismatch false failures on unmodified exports

Recommendation:
**Accepted** per team direction for SKSH-319. Optional follow-up (out of scope): remove unused `rowMetadataById`/`uploadedBy` preparation or restore canonical read-only checks if product later requires explicit errors.

**PR comment:** Not required (accepted).

---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Read-only CSV display columns not validated on import (intentional) | HIGH | Accepted | src/utils/video-library-import.utils.ts | 84-96 | — |

**Merge readiness:** **Merge-ready** — no open Critical/High blockers (1 High accepted).
