# Backend PR Review — skillshow (`SKSH-245`)

**Repo:** skillshow — `https://github.com/fx31labs-mvp/skillshow.git`  
**Branch:** `SKSH-245`  
**Base:** `main...HEAD` @ `2890a85`  
**Initial review:** 2026-06-12  
**Re-review:** 2026-06-12 @ `2890a85` (`3e88049 fix; pr changes`)  
**Scope:** Separate edit-request source uploads (`isEditUploaded`) from athlete My Videos; promote approved edit outputs to Skill Show Uploaded tab (`isSkillshowUploaded`)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [frontend.md](./frontend.md)

**Findings:** 2 (0 Critical, 2 High) — **2 Fixed**

### Protected modules

| Module | Status |
|--------|--------|
| `list-query.validation.ts`, `list-query-aggregation.utils.ts`, audit-log modules, change-stream modules | **Not modified** |

### Files reviewed

| File | Change |
|------|--------|
| `src/constants/video.constants.ts` | `VIDEO_EXCLUDE_EDIT_UPLOADED_QUERY`, `VIDEO_SKILLSHOW_UPLOADED_MATCH`, `VIDEO_SKILL_SHOW_LIST_MATCH` |
| `src/models/video.model.ts` | `isEditUploaded`, `isSkillshowUploaded` fields + partial unique index `(user, key)` |
| `src/repositories/video.repository.ts` | `findSkillshowUploadedIdByUserAndKey`, `upsertSkillshowUploadedVideo` |
| `src/services/edit-request-output.service.ts` | Promote-before-approve; recovery for `APPROVED` without promoted row |
| `src/services/video-upload.service.ts` | Mark edit-request `recordVideo` rows as `isEditUploaded: true` |
| `src/swagger/video.swagger.ts` | Swagger fields for new flags |
| `src/types/edit-request-output/participant-output.types.ts` | Optional `videoId` on approve DTO |
| `src/types/video-query.types.ts` | `CreateSkillshowUploadedVideoInput` |
| `src/types/video.types.ts` | Response DTO flags |
| `src/utils/video-user-scope.utils.ts` | My Videos tab scope excludes edit uploads; includes promoted outputs |
| `src/utils/video.utils.ts` | Browser/distribution list filters + response mapping |
| `tests/utils/video-user-scope.utils.test.ts` | Scope query expectations updated |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Shared Mongo filters centralized in `video.constants.ts` | ✅ DRY |
| `buildVideoBrowserListFilter`, `buildVideoDistributionListFilter`, `buildAthleteUploadedVideoScopeQuery`, `buildMyVideosListScopeQuery` all consume shared constants | ✅ Global consistency |
| `upsertSkillshowUploadedVideo` single create path + duplicate-key handling | ✅ |
| `buildParticipantVersionDtoWithVideoId` extracts repeated return mapping | ✅ DRY |
| Promote-before-approve + `APPROVED` recovery path | ✅ Fixed (#1) |
| Partial unique index + `isDuplicateKeyError` upsert | ✅ Fixed (#2) |
| Protected list/audit modules untouched | ✅ |

### Positive notes

- **Re-review fixes:** `3e88049` addresses both prior High findings — promotion runs before persisting approval; concurrent creates are guarded by a partial unique index and duplicate-key fallback.
- **Idempotent recovery:** If a version is already `APPROVED` but promotion was missing (legacy partial failure), approve re-runs promotion or returns the existing `videoId` without throwing `INVALID_TRANSITION`.
- **Clear separation of concerns:** Edit-request source uploads stay in Mongo for edit-request linkage but are excluded from My Videos and the edit-request video browser via `VIDEO_EXCLUDE_EDIT_UPLOADED_QUERY`.
- **Cross-stack types:** Swagger + `VideoResponseDto` + participant approve DTO expose the new flags/`videoId` for the admin UI.

---

## GitHub comments

No new inline comments — prior High findings resolved in `3e88049`.

---

## Findings

---
Approve persisted before video promotion with no recovery path

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: skillshow/src/services/edit-request-output.service.ts  
Lines: 1547-1616

Description:
**KISS / operational correctness (initial review).** Original diff saved approval before calling `promoteApprovedVersionToAthleteVideo`, leaving no retry path on promotion failure.

**Re-review evidence:** Happy path now promotes first (line 1575), then marks the version `APPROVED` and saves (1582–1588). Recovery branch handles `reviewStatus === APPROVED` without a promoted row (1547–1565) or returns existing `videoId` when promotion already succeeded (1548–1553).

Impact (was):
- Athlete approved but video missing from My Videos with no retry.

Recommendation (applied):
Promote before approve persistence; add idempotent recovery when already `APPROVED`.

---

---
Promoted video create is not race-safe

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: skillshow/src/repositories/video.repository.ts  
Lines: 80-111

Description:
**DRY / data integrity (initial review).** Original find-then-create had no unique constraint; concurrent approve could insert duplicates.

**Re-review evidence:**
- Partial unique index on `{ user: 1, key: 1 }` with `partialFilterExpression: { isSkillshowUploaded: true }` in `video.model.ts` (184–190).
- `upsertSkillshowUploadedVideo` catches `isDuplicateKeyError` and returns the existing `_id` (80–111).

Impact (was):
- Duplicate My Videos rows for one approved output under concurrent requests.

Recommendation (applied):
Partial unique index + duplicate-key fallback on create.

---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Approve persisted before video promotion with no recovery path | HIGH | ✅ Fixed | skillshow/src/services/edit-request-output.service.ts | 1547-1616 | — |
| 2 | Promoted video create is not race-safe | HIGH | ✅ Fixed | skillshow/src/repositories/video.repository.ts | 80-111 | — |

**Merge readiness:** **Merge-ready** — all findings Fixed; frontend report also clear ([frontend.md](./frontend.md)).
