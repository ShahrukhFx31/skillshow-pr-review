# PR review — SKSH-378 (skillshow)

| Field | Value |
|-------|-------|
| PR | [#225](https://github.com/SkillshowFx/skillshow/pull/225) |
| Branch | `SKSH-378` → `main` |
| Head | `SKSH-378` @ `1110e9e7` |
| Scope | S3 upload key safety + multipart follow-up contract; fix “same-name approved video” promoted storage collisions |
| Prompt | `pr-review/prompts/backend-system-prompt.md` |

## GitHub comments

_No open inline findings._

## Findings

---
Multipart upload must reuse server-returned key (previous blocker now fixed)

Risk Level: CRITICAL
File Path: src/controllers/upload.controller.ts
Lines: 18-34, 75-106, 133-146

Description:
Previously this PR had a risk of regenerating a new UUID key on each multipart step if only `fileName` was provided, breaking multipart integrity. The current diff fixes this by requiring `key` on multipart follow-up steps and validating it via `s3Service.isUploadKeyAllowedForUser` before issuing part URLs / completing / aborting.

Impact:
- Prevents multipart uploads from breaking due to key mismatch across steps.
- Improves security by enforcing user-scoped key validation for multipart follow-ups (**Security / S3**).

Recommendation:
✅ Fixed as implemented: keep `resolveMultipartUploadKey()` and ensure routes stay wired to `multipart*BodySchema` validators so clients must send `key`.
---

---
Thumbnail is copied when promoted video key is re-scoped (previous blocker now fixed)

Risk Level: HIGH
File Path: src/services/edit-request-output.service.ts
Lines: 1496-1650 (approx; new helpers `resolvePromotedSkillshowVideoStorage` / `findPromotedSkillshowVideoId`)

Description:
Previously, promoted outputs could end up reusing the same underlying S3 key (same filename), causing collisions; and thumbnail could remain pointing at the shared/editor key. The current diff resolves this by:
- Detecting collisions for the athlete+key mapping.
- Copying the source object to an athlete+version-scoped key when needed.
- Re-scoping/copying thumbnails alongside the main copy.

Impact:
- Prevents “same-name approved video” overwrites/collisions.
- Ensures promoted videos and thumbnails remain correctly associated to the athlete/version.

Recommendation:
✅ Fixed as implemented: keep the collision check + copy flow and associated tests.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Multipart upload must reuse server-returned key (previous blocker now fixed) | CRITICAL | ✅ Fixed | src/controllers/upload.controller.ts | 18-34, 75-106, 133-146 |
| 2 | Thumbnail is copied when promoted video key is re-scoped (previous blocker now fixed) | HIGH | ✅ Fixed | src/services/edit-request-output.service.ts | ~1496-1650 |

**Merge readiness:** No open Critical/High blockers. The diff addresses prior concerns by enforcing user-scoped `key` for multipart follow-ups and by re-scoping/copying promoted outputs (and thumbnails) to avoid collisions when filenames repeat.
