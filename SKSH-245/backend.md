# Backend PR Review — skillshow (`SKSH-245`)

**Repo:** skillshow — `https://github.com/fx31labs-mvp/skillshow.git`  
**Branch:** `SKSH-245`  
**Base:** `main...HEAD` @ `64d6e3a`  
**Initial review:** 2026-06-12  
**Scope:** Separate edit-request source uploads (`isEditUploaded`) from athlete My Videos; promote approved edit outputs to Skill Show Uploaded tab (`isSkillshowUploaded`)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [frontend.md](./frontend.md)

**Findings:** 2 (0 Critical, 2 High) — **2 Open**

### Protected modules

| Module | Status |
|--------|--------|
| `list-query.validation.ts`, `list-query-aggregation.utils.ts`, audit-log modules, change-stream modules | **Not modified** |

### Files reviewed

| File | Change |
|------|--------|
| `src/constants/video.constants.ts` | `VIDEO_EXCLUDE_EDIT_UPLOADED_QUERY`, `VIDEO_SKILLSHOW_UPLOADED_MATCH`, `VIDEO_SKILL_SHOW_LIST_MATCH` |
| `src/models/video.model.ts` | `isEditUploaded`, `isSkillshowUploaded` schema fields + indexes |
| `src/repositories/video.repository.ts` | `findSkillshowUploadedIdByUserAndKey`, `createSkillshowUploadedVideo` |
| `src/services/edit-request-output.service.ts` | `promoteApprovedVersionToAthleteVideo`; approve returns `videoId` |
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
| `createSkillshowUploadedVideo` colocated in `video.repository` (single create path) | ✅ |
| `{ user, ...VIDEO_SKILLSHOW_UPLOADED_MATCH }` repeated inline 3× in `buildMyVideosListScopeQuery` | ⚠️ Minor DRY — acceptable at this size |
| Approve persistence runs before promotion with no transaction / retry path | ❌ See #1 |
| Idempotency via find-then-create without unique constraint | ❌ See #2 |
| Protected list/audit modules untouched | ✅ |

### Positive notes

- **Clear separation of concerns:** Edit-request source uploads stay in Mongo for edit-request linkage but are excluded from My Videos and the edit-request video browser via `VIDEO_EXCLUDE_EDIT_UPLOADED_QUERY`.
- **Skill Show tab union:** Promoted outputs (`isSkillshowUploaded`) appear alongside library-tagged rows using `VIDEO_SKILL_SHOW_LIST_MATCH` in distribution list filters and `$or` scopes in `buildMyVideosListScopeQuery`.
- **Cross-stack types:** Swagger + `VideoResponseDto` + participant approve DTO expose the new flags/`videoId` for the admin UI.
- **Scope tests updated:** `video-user-scope.utils.test.ts` covers athlete, skillshow-only, and combined tab filters.

---

## GitHub comments

### 1. `src/services/edit-request-output.service.ts` line 1570

**PR comment (line 1570):** **High:** This promotion runs after the version and edit-request doc are already saved as approved (lines 1544–1557 above are unchanged but execute first). If `promoteApprovedVersionToAthleteVideo` throws here, the version stays `APPROVED` and a retry hits `INVALID_TRANSITION:approve_version:already_reviewed`, leaving the athlete without a My Videos row. Wrap approve + promote in a Mongo transaction, promote before marking approved, or add a recovery path when `reviewStatus === APPROVED` but no `isSkillshowUploaded` row exists for `version.key`.

### 2. `src/repositories/video.repository.ts` line 78

**PR comment (line 78):** **High:** `findSkillshowUploadedIdByUserAndKey` + `createSkillshowUploadedVideo` is check-then-create with no unique index on `(user, key, isSkillshowUploaded)`. Concurrent approve requests (double-click / retry) can both pass the find and insert duplicate promoted videos for the same S3 key. Add a partial unique index and use upsert/`findOneAndUpdate` with `upsert: true`, or catch duplicate-key and return the existing `_id`.

---

## Findings

---
Approve persisted before video promotion with no recovery path

Risk Level: HIGH  
**Status:** Open  
File Path: skillshow/src/services/edit-request-output.service.ts  
Lines: 1570-1574

Description:
**KISS / operational correctness.** `approveVersionForParticipant` saves the version as `APPROVED`, persists edit-request status/history, emits realtime/notification, and only then calls `promoteApprovedVersionToAthleteVideo`. Promotion is not wrapped in a transaction with the approval writes. A failure in `createSkillshowUploadedVideo` (Mongo, validation, transient error) leaves the version permanently approved while no `isSkillshowUploaded` video exists.

Because re-approve is blocked:

```1530:1535:skillshow/src/services/edit-request-output.service.ts
    if (
      version.reviewStatus !==
      EDIT_REQUEST_VERSION_REVIEW_STATUS.WAITING_FOR_REVIEW
    ) {
      throw new Error("INVALID_TRANSITION:approve_version:already_reviewed");
    }
```

there is no client-visible way to retry promotion for that version.

Impact:
- Athlete sees an approved output in the edit request but the video never appears under My Videos → Skill Show Uploaded.
- Support/manual DB intervention required to backfill the promoted row.
- Frontend toast (“now available in My Videos…”) may show even when promotion failed (500 after partial commit).

Recommendation:
Pick one of:
1. **Transaction:** Wrap version approval, edit-request save, and `createSkillshowUploadedVideo` in a Mongo session transaction; roll back on promotion failure.
2. **Order swap:** Promote first; only mark the version approved after a successful create (rollback promotion on downstream failure if needed).
3. **Idempotent recovery:** At the start of approve (or a dedicated repair job), if `reviewStatus === APPROVED` and no promoted row exists for `version.key`, run promotion and return `videoId` without throwing `already_reviewed`.

**PR comment (`edit-request-output.service.ts` line 1570):** **High:** This promotion runs after the version and edit-request doc are already saved as approved. If `promoteApprovedVersionToAthleteVideo` throws here, the version stays `APPROVED` and a retry hits `INVALID_TRANSITION`. Wrap approve + promote in a transaction, promote before marking approved, or add recovery when approved but no promoted video exists.

---

---
Promoted video create is not race-safe

Risk Level: HIGH  
**Status:** Open  
File Path: skillshow/src/repositories/video.repository.ts  
Lines: 63-93

Description:
**DRY / data integrity.** `promoteApprovedVersionToAthleteVideo` uses find-then-create idempotency:

```1491:1496:skillshow/src/services/edit-request-output.service.ts
    const existingId =
      await videoRepository.findSkillshowUploadedIdByUserAndKey(
        athleteOid,
        version.key
      );
    if (existingId) return existingId;
```

There is no partial unique index on `{ user: 1, key: 1 }` (or equivalent) scoped to `isSkillshowUploaded: true`. Two concurrent approve calls for the same version can both observe `null` and call `VideoModel.create`, producing duplicate promoted documents referencing the same S3 key.

Impact:
- Duplicate My Videos rows for one approved output.
- Ambiguous `_id` for distribution, delete, and audit; downstream logic may attach jobs to different duplicates.
- Idempotency intent is undermined under concurrent requests.

Recommendation:
Add a sparse/partial unique compound index, e.g. `{ user: 1, key: 1 }` with `partialFilterExpression: { isSkillshowUploaded: true }`, and replace create with:

```typescript
const doc = await VideoModel.findOneAndUpdate(
  { user, key, isSkillshowUploaded: true },
  { $setOnInsert: { /* fields */ } },
  { upsert: true, new: true }
);
return String(doc._id);
```

Alternatively catch Mongo duplicate-key (`E11000`) and re-query `findSkillshowUploadedIdByUserAndKey`.

**PR comment (`video.repository.ts` line 78):** **High:** Check-then-create without a unique index can insert duplicate promoted videos on concurrent approve. Add a partial unique index on `(user, key, isSkillshowUploaded)` and use upsert or duplicate-key handling.

---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Approve persisted before video promotion with no recovery path | HIGH | Open | skillshow/src/services/edit-request-output.service.ts | 1570-1574 | 1570 |
| 2 | Promoted video create is not race-safe | HIGH | Open | skillshow/src/repositories/video.repository.ts | 63-93 | 78 |

**Merge readiness:** **Not merge-ready** — 2 open High findings (stuck approve state + duplicate promoted rows under concurrency). Fix before merge.
