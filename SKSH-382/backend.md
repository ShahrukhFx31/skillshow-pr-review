# PR review (SKSH-382) — skillshow

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow` |
| PR | [#242](https://github.com/SkillshowFx/skillshow/pull/242) |
| Branch | `SKSH-382` → `main` |
| Head | `27877fae7502ba52d779a5dd168bdf5a29833ba2` |
| Scope | Video Library bulk-upload validate; register overwrite; 10 GB multipart cap |
| Prompts | `pr-review/prompts/backend-system-prompt.md`, `pr-review/SECURITY-AUDIT-PRE-RELEASE.md` |

### Protected modules

| Module | Status |
|--------|--------|
| List-query / audit / change-stream protected modules | **Not modified** ✅ |

### Positive notes

- New `POST /bulk-upload/validate` is gated with `authorize({ roles: ["admin"] })`.
- Filename parsing, Joi batch max (100), and unit tests cover the happy path well.
- Paired FE correctly registers with `parsed.eventSlug` (not top-level `eventId`).

## GitHub comments

### `src/repositories/user.repository.ts`
- **L685** — Validate user lookup omits `isActive` / athlete-role checks (HIGH)

### `src/services/video-library-bulk-upload.service.ts`
- **L126** — Validate `eventId` is `EV{n}` while register expects slug (HIGH)
- **L139** — No within-batch duplicate detection (HIGH)

### `src/validation/video-library.validation.ts`
- **L74** — `sizeBytes` unbounded in Joi schema (HIGH)

## Findings

---
Validate accepts inactive / non-athlete users that register rejects

Risk Level: HIGH
File Path: src/repositories/user.repository.ts
Lines: 685-688

Description:
**Contract** — `findLeanActiveUsersBySeqs` only filters `isDeleted`; it does not require `isActive: true` or the athlete role. Register uses `athleteUsernamesExist` (`isActive: true` + athlete role). Parent/coach/inactive seqs pass validate then fail register with `"Invalid athlete selection"`.

Impact:
- False-green validate → failed register after large S3 uploads
- Wasted bandwidth on up to 10 GB files

Recommendation:
Align validate with register: filter `isActive: true` and athlete role (or the same repository helper keyed by seq). Add tests for inactive and non-athlete seqs.
---

---
Validate response `eventId` is public `EV{n}`; register expects event slug

Risk Level: HIGH
File Path: src/services/video-library-bulk-upload.service.ts
Lines: 124-128

Description:
**Contract** — Top-level `eventId` is set to `` `${EVENT_ID_PREFIX}${parsed.eventSeq}` `` (e.g. `EV2000`). Register `body.eventId` must be the platform **slug** (`parsed.eventSlug`). Same field name, different semantics across endpoints.

Impact:
- Any client mapping validate `eventId` → register `eventId` gets `"Invalid event"`
- Footgun even though the paired admin UI uses `parsed.eventSlug`

Recommendation:
Expose display id as `publicEventId` / `eventSeqLabel` and top-level `eventSlug` for register, or drop top-level `eventId` and document `parsed.eventSlug` only.
---

---
Within-batch duplicates not detected

Risk Level: HIGH
File Path: src/services/video-library-bulk-upload.service.ts
Lines: 139-145

Description:
**Contract** — `findDuplicate` only compares against existing DB rows. Two identical files in one batch both get `valid`.

Impact:
- Duplicate library rows from a single bulk upload

Recommendation:
Track a Set of normalized keys (`eventSlug|usernames|title`) within the batch and mark later collisions as `duplicate`.
---

---
Bulk validate Joi allows unbounded `sizeBytes`

Risk Level: HIGH
File Path: src/validation/video-library.validation.ts
Lines: 74

Description:
**Contract** — Schema only requires positive `sizeBytes`; 10 GB cap is service-only. Multipart start schema in this PR correctly uses `.max(...)`.

Impact:
- Oversized payloads pass middleware; inconsistent error shape vs service

Recommendation:
`.max(VIDEO_LIBRARY_BULK_UPLOAD_MAX_FILE_SIZE_BYTES)` with the existing `FILE_TOO_LARGE` message.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Validate accepts inactive / non-athlete users that register rejects | HIGH | Open | `src/repositories/user.repository.ts` | 685-688 |
| 2 | Validate `eventId` is `EV{n}`; register expects slug | HIGH | Open | `src/services/video-library-bulk-upload.service.ts` | 124-128 |
| 3 | Within-batch duplicates not detected | HIGH | Open | `src/services/video-library-bulk-upload.service.ts` | 139-145 |
| 4 | Bulk validate Joi allows unbounded `sizeBytes` | HIGH | Open | `src/validation/video-library.validation.ts` | 74 |

**Merge readiness:** Not ready — 4 open High validate→register / batch contract issues.
