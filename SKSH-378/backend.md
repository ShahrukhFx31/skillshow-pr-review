# PR review (SKSH-378) — skillshow

PR: `https://github.com/SkillshowFx/skillshow/pull/225`  
Base: `main`  
Head: `SKSH-378` @ `761e7cac`

Prompt: `pr-review/prompts/backend-system-prompt.md`

## GitHub comments

### `src/controllers/upload.controller.ts`
- **CRITICAL** — Multipart upload regenerates a new UUID key on every step (line 22)

### `src/services/edit-request-output.service.ts`
- **HIGH** — Thumbnail may not be copied when main video key is re-scoped (summary-only; see finding #2)

## Findings

---
Multipart upload regenerates a new UUID key on every step

Risk Level: CRITICAL  
File Path: src/controllers/upload.controller.ts  
Lines: 21-23

Description:
**Contract / operational correctness.** `uploadKeyForUser` now calls `uniqueKeyFromUserFileName`, which embeds a new `randomUUID()` on every invocation. Multipart endpoints (`start`, `presigned-urls`, `complete`, `abort`) derive the S3 key from `fileName` on each request and do not accept the `key` returned by `start`.

The admin UI (`uploadService.ts`, `s3-file-upload.utils.ts`) only passes `fileName` + `uploadId` after start; it keeps `key` for `recordUpload` but not for subsequent multipart calls.

Impact:
- Part presigned URLs target a different object than the multipart upload started with.
- `completeMultipartUpload` finalizes yet another key.
- Large file uploads (>10 MB) fail or leave orphaned multipart state in S3.
- Affects general upload routes and any flow using shared multipart helpers (e.g. edit-request output uploads).

Recommendation:
Do **not** use `uniqueKeyFromUserFileName` for multipart steps that re-derive the key from `fileName`. Prefer one of:
1. Add required `key` to multipart request bodies (client passes the `key` from `start`), **or**
2. Keep `keyFromUserFileName` for multipart-only paths and use `uniqueKeyFromUserFileName` only for single-shot presigned uploads where the returned `key` is carried through the whole flow.

Add an integration test that asserts `start` → `presigned-urls` → `complete` use the same key when mocks return distinct UUIDs.

---

PR comment (inline):
`uniqueKeyFromUserFileName` generates a new UUID per call, but multipart handlers invoke `uploadKeyForUser(fileName)` on every step while clients only send `fileName` + `uploadId`. That breaks the multipart pipeline (parts upload to a different key than `start`/`complete`). Require `key` on multipart bodies or reserve unique keys for single-shot presigned uploads only.

---
Thumbnail may not be copied when main video key is re-scoped

Risk Level: HIGH  
File Path: src/services/edit-request-output.service.ts  
Lines: 1543-1558 (approx. in PR head)

Description:
**Operational correctness.** When `resolvePromotedSkillshowVideoStorage` copies the main video to a version-scoped key due to `(user, key)` collision, thumbnail re-scoping only runs when `findSkillshowUploadedIdByUserAndKey` finds an existing **video** row whose `key` equals `version.thumbnailKey`. Thumbnail keys are rarely stored as video `key`, so promoted outputs often keep the editor's thumbnail S3 path while the video object was copied under the athlete namespace.

Impact:
- Second+ approved outputs with the same source filename can reference a thumbnail object outside the athlete-scoped promoted key layout.
- If the editor/source thumbnail is deleted or permissions change, the athlete-facing promoted video may lose its thumbnail while the main video plays.

Recommendation:
When copying the main object to `versionScopedKey`, also copy the thumbnail to `skillshowPromotedKeyFromUserVersion(..., versionId + '-thumb', ...)` whenever `version.thumbnailKey` is present (mirror main-key collision handling), not only when a video row already exists for the thumbnail key.

---

### Positive notes

- **Root cause fix for duplicate promotions:** Version-scoped athlete keys + `findPromotedSkillshowVideoId` correctly handle multiple approved outputs sharing the same source S3 key.
- **S3 safety:** `skillshowPromotedKeyFromUserVersion` stays under `uploads/{userId}/`; `copyObject` is internal to approve flow.
- **Single-shot uploads:** `getPresignedUrl` / `recordUpload` / `video-upload.service` correctly return and reuse one unique key.
- **Tests:** Good coverage for collision vs non-collision promote paths and `uniqueKeyFromUserFileName` behavior.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Multipart upload regenerates a new UUID key on every step | CRITICAL | Open | `src/controllers/upload.controller.ts` | 21-23 |
| 2 | Thumbnail may not be copied when main video key is re-scoped | HIGH | Open | `src/services/edit-request-output.service.ts` | ~1543-1558 |

**Merge readiness:** Open Critical/High blockers — multipart upload regression must be fixed before merge.
