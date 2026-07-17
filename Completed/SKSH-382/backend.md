# PR review (SKSH-382) — skillshow

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow` |
| PR | [#242](https://github.com/SkillshowFx/skillshow/pull/242) |
| Branch | `SKSH-382` → `main` |
| Head | `69f79d178cfe5605e50a0b6de7f9f84733693c1d` |
| Scope | Video Library bulk-upload validate; register overwrite; 10 GB multipart cap |
| Prompts | `pr-review/prompts/backend-system-prompt.md`, `pr-review/SECURITY-AUDIT-PRE-RELEASE.md` |
| Verified | 2026-07-17 vs prior head `27877fae` |

### Protected modules

| Module | Status |
|--------|--------|
| List-query / audit / change-stream protected modules | **Not modified** ✅ |

### Positive notes

- New `POST /bulk-upload/validate` is gated with `authorize({ roles: ["admin"] })`.
- Filename parsing, Joi batch max (100), and unit tests cover the happy path well.
- Paired FE correctly registers with `eventSlug` / `parsed.eventSlug` (not display `EV{n}`).
- Follow-ups: `isActive` + username on validate lookup; top-level `eventSlug`; within-batch dup keys; Joi `sizeBytes` max.

## GitHub comments

No open GitHub inline comments (prior REQUEST_CHANGES findings resolved on head).

## Findings

---
Validate accepts inactive / non-athlete users that register rejects

Risk Level: HIGH
File Path: src/repositories/user.repository.ts
Lines: 685-688

Description:
**Contract** — `findLeanActiveUsersBySeqs` only filtered `isDeleted`; register used athlete role.

Impact:
- False-green validate → failed register after large S3 uploads

Recommendation:
Align validate with register: filter `isActive: true` and athlete role (or the same repository helper keyed by seq).

Status: ✅ Fixed — `isActive: true` + non-empty username on validate; register `athleteUsernamesExist` no longer requires athlete role (same active+username gate).
---

---
Validate response `eventId` is public `EV{n}`; register expects event slug

Risk Level: HIGH
File Path: src/services/video-library-bulk-upload.service.ts
Lines: 124-128

Description:
**Contract** — Top-level `eventId` was display `EV{n}` while register expected slug.

Impact:
- Clients mapping validate `eventId` → register got `"Invalid event"`

Recommendation:
Expose `eventSlug` for register; keep display id separate.

Status: ✅ Fixed — top-level `eventSlug` (+ comment that register must use it); FE maps `result.eventSlug`.
---

---
Within-batch duplicates not detected

Risk Level: HIGH
File Path: src/services/video-library-bulk-upload.service.ts
Lines: 139-145

Description:
**Contract** — `findDuplicate` only compared against DB rows.

Impact:
- Duplicate library rows from a single bulk upload

Recommendation:
Track a Set of normalized keys within the batch.

Status: ✅ Fixed — `seenBatchKeys` + `buildBatchKey` + `DUPLICATE_IN_BATCH` tests.
---

---
Bulk validate Joi allows unbounded `sizeBytes`

Risk Level: HIGH
File Path: src/validation/video-library.validation.ts
Lines: 74

Description:
**Contract** — Schema only required positive `sizeBytes`; 10 GB cap was service-only.

Impact:
- Oversized payloads passed middleware; inconsistent error shape

Recommendation:
`.max(VIDEO_LIBRARY_BULK_UPLOAD_MAX_FILE_SIZE_BYTES)`.

Status: ✅ Fixed — Joi `.max(...)` with `FILE_TOO_LARGE` message.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Validate accepts inactive / non-athlete users that register rejects | HIGH | ✅ Fixed | `src/repositories/user.repository.ts` | 685-688 |
| 2 | Validate `eventId` is `EV{n}`; register expects slug | HIGH | ✅ Fixed | `src/services/video-library-bulk-upload.service.ts` | 124-128 |
| 3 | Within-batch duplicates not detected | HIGH | ✅ Fixed | `src/services/video-library-bulk-upload.service.ts` | 139-145 |
| 4 | Bulk validate Joi allows unbounded `sizeBytes` | HIGH | ✅ Fixed | `src/validation/video-library.validation.ts` | 74 |

**Merge readiness:** Ready — prior High findings fixed on `69f79d17`; no new Critical/High/Medium open.
