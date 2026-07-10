# PR review — SKSH-385 (backend)

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow` |
| PR | [#233](https://github.com/SkillshowFx/skillshow/pull/233) |
| Branch | `SKSH-385` → `main` |
| Head | `aa1d221c6eeb7b68017f348122e602177de06c46` |
| Scope | Delete S3 objects when a Video Library row is soft-deleted; shared `collectVideoObjectKeys` helper; published-distribution guard |
| Prompts | `pr-review/prompts/backend-system-prompt.md`, `SECURITY-AUDIT-PRE-RELEASE.md` |

## GitHub comments

_No open findings._

## Findings

---
Library delete skips published-distribution guard before S3 removal

Risk Level: HIGH
File Path: src/services/video-library.service.ts
Lines: 329-337

Description:
**Global consistency** / **Contract** — `VideoUploadService.deleteUploadedVideo` blocks deletion when `vendorLogRepository.existsCompletedForVideo(video._id)` is true (`HAS_PUBLISHED_DISTRIBUTION`), because removing the underlying S3 object breaks live platform posts. `VideoLibraryService.deleteItem` initially deleted all collected S3 keys after soft-delete with no equivalent check.

Impact:
- Published social posts could point at deleted S3 keys while distribution history still exists.
- Inconsistent delete safety between athlete uploads and admin library removal.

Recommendation:
Reuse the athlete-delete guard before mutating the row or deleting S3 objects; wire `respondIfAppError` in the controller; add a service test.

**Re-verify:** ✅ Fixed — `existsCompletedForVideo` guard throws `BadRequestError(HAS_PUBLISHED_DISTRIBUTION)` before soft-delete/S3 cleanup; controller uses `respondIfAppError`; test asserts row stays `active` and S3 is not called.
---

## Positive notes

- `collectVideoObjectKeys` deduplicates main video, thumbnail, history, frame, and per-platform keys — broader cleanup than the athlete delete path.
- S3 deletion runs after Mongo soft-delete with per-key error logging, matching the established "do not block DB delete on S3 failure" posture.
- Tests cover single-key, multi-key S3 cleanup, and published-distribution rejection.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Library delete skips published-distribution guard before S3 removal | HIGH | ✅ Fixed | src/services/video-library.service.ts | 329-337 |

**Merge readiness:** No open Critical/High/Medium blockers.
