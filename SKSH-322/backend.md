# Backend PR Review — skillshow (`SKSH-322`)

**Repo:** skillshow  
**Branch:** `SKSH-322`  
**Base:** `main...HEAD` @ `6ef2c04`  
**Scope:** Public profile sharing (`GET/PATCH /v1/profile/settings`, `GET /api/public/profiles/:shareToken`), user model share token, paginated public videos, video-share DTO refactor (Critical / High)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 2 in scope (0 Critical, 2 High) — **2 Open**

### Protected modules

| Module | Status |
|--------|--------|
| `list-query.validation.ts`, `list-query-aggregation.utils.ts`, audit-log stack, change-stream modules | **Not modified** ✅ |

### Files reviewed (28 changed)

Profile settings controller/routes/validation/swagger, public profile controller/routes/service, `profile.service` settings + share helpers, `user.repository` visibility fields, `video.repository` profile video queries, `video-share.utils` DTO mapper, `video-public.service` refactor, user model fields, tests.

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| `mapLeanPublicShareVideoRowToDto` shared by video + profile share surfaces | ✅ Good DRY |
| `profile-share.utils` / `video-share.utils` separation | ✅ Clear |
| Profile settings validation via Joi + `validate()` middleware | ✅ |
| Public profile list filter vs video share filter | ⚠️ Mismatch — see #2 |
| Controller reads `validatedQuery` after query validation | ⚠️ Violation — see #1 |
| Token generation with collision retry | ✅ |
| Tests for settings, public service, share utils | ✅ Good coverage |
| Protected list/audit modules untouched | ✅ |

---

## GitHub comments (Open findings)

### 1. `src/controllers/public-profile.controller.ts` line 14

**PR comment (line 14):** **High (Contract):** `validate(publicProfileShareQuerySchema, "query")` sets `req.validatedQuery`, but this handler reads raw `req.query` for `page`. Use `(req as ValidatedQueryRequest).validatedQuery` (same pattern as list controllers) so coerced/defaulted values are the single source of truth.

### 2. `src/repositories/video.repository.ts` line 220

**PR comment (line 220):** **High (Global consistency / Contract):** `publicProfileVideosFilter` spreads `VIDEO_PUBLIC_ONLY_QUERY` (`isPublic: true`), but `findPublicShareLean` / share links use `VIDEO_PUBLIC_SHARE_QUERY` (also excludes `inVideoLibrary`). Public-profile grid links to `/share/video/:id` — library rows that are public can appear in the profile list yet 404 on click. Reuse `VIDEO_PUBLIC_SHARE_QUERY` here (or a shared helper) so list, count, and share endpoints agree.

---

## Findings

---
Public profile controller bypasses validatedQuery

Risk Level: HIGH  
File Path: src/controllers/public-profile.controller.ts  
Lines: 13-17

Description:
**Contract.** `public-profile.routes.ts` wires `validate(publicProfileShareQuerySchema, "query")`, which stores the coerced Joi result on `req.validatedQuery` (`validate.middleware.ts` documents that controllers must read query params from there only). `getShareView` instead parses `req.query.page` directly.

Impact:
- Diverges from established list/public query handling and can drift if the schema gains coercion or new fields
- Future schema changes (e.g. `max` page cap) would not apply to the controller path

Recommendation:
Read validated query params:

```typescript
import type { ValidatedQueryRequest } from "../types/common";

const { page = 1 } = ((req as ValidatedQueryRequest).validatedQuery ?? {}) as {
  page?: number;
};
const data = await profilePublicService.getPublicProfileShareView(id, page);
```

**PR comment (line 14):** **High (Contract):** Use `req.validatedQuery.page` after `validate(..., "query")` — do not read raw `req.query`.

---

---
Profile public video filter inconsistent with video share eligibility

Risk Level: HIGH  
File Path: src/repositories/video.repository.ts  
Lines: 220-224

Description:
**Global consistency / Contract.** `publicProfileVideosFilter` uses `VIDEO_PUBLIC_ONLY_QUERY` (`{ isPublic: true }`). Individual public share pages and view-count increments use `VIDEO_PUBLIC_SHARE_QUERY`, which additionally requires `inVideoLibrary: { $ne: true }`. The frontend profile grid links each card to `/share/video/${video.id}`, which hits `findPublicShareLean` with the stricter filter.

Impact:
- Public video-library rows marked `isPublic: true` can appear on the profile share page but return 404 when opened
- `pagination.total` / video-count badge can over-count relative to actually shareable videos

Recommendation:
Align the profile list with share eligibility:

```typescript
private publicProfileVideosFilter(userId: string): Record<string, unknown> {
  return {
    user: this.toObjectId(userId),
    ...VIDEO_PUBLIC_SHARE_QUERY,
  };
}
```

Add/adjust a repository test asserting library public rows are excluded from profile pagination.

**PR comment (line 220):** **High:** Use `VIDEO_PUBLIC_SHARE_QUERY` in `publicProfileVideosFilter` so profile grid videos match `/api/public/videos/:id` eligibility.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Public profile controller bypasses validatedQuery | HIGH | Open | src/controllers/public-profile.controller.ts | 13-17 |
| 2 | Profile public video filter inconsistent with video share eligibility | HIGH | Open | src/repositories/video.repository.ts | 220-224 |

**Merge readiness:** **Not merge-ready** — 2 open High findings (contract + share-eligibility mismatch). Fix before merge; frontend depends on consistent public video filtering for profile grid links.
