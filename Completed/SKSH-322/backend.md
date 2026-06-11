# Backend PR Review — skillshow (`SKSH-322`)

**Repo:** skillshow  
**Branch:** `SKSH-322`  
**Base:** `main...HEAD` @ `0ac9a95`  
**Initial review:** 2026-06-11 @ `6ef2c04`  
**Re-review:** 2026-06-11 @ `0ac9a95` (`feat: enhance public profile sharing functionality with query validation and repository updates`)  
**Scope:** Public profile sharing (`GET/PATCH /v1/profile/settings`, `GET /api/public/profiles/:shareToken`), user model share token, paginated public videos, video-share DTO refactor (Critical / High)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 2 (0 Critical, 2 High) — **0 Open**, **2 Fixed**

### Protected modules

| Module | Status |
|--------|--------|
| `list-query.validation.ts`, `list-query-aggregation.utils.ts`, audit-log stack, change-stream modules | **Not modified** ✅ |

### Files reviewed (29 changed)

Profile settings controller/routes/validation/swagger, public profile controller/routes/service, `profile.service` settings + share helpers, `user.repository` visibility fields, `video.repository` profile video queries, `video-share.utils` DTO mapper, `video-public.service` refactor, user model fields, tests (including new `video.repository.test.ts`).

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| `mapLeanPublicShareVideoRowToDto` shared by video + profile share surfaces | ✅ Good DRY |
| `profile-share.utils` / `video-share.utils` separation | ✅ Clear |
| Profile settings validation via Joi + `validate()` middleware | ✅ |
| Public profile list filter vs video share filter | ✅ Fixed — `VIDEO_PUBLIC_SHARE_QUERY` |
| Controller reads `validatedQuery` after query validation | ✅ Fixed |
| Token generation with collision retry | ✅ |
| Repository tests for share-eligibility filter | ✅ New |
| Protected list/audit modules untouched | ✅ |

### Positive notes

- Re-review commit addresses both prior High findings with typed `PublicProfileShareQuery` and repository tests asserting `VIDEO_PUBLIC_SHARE_QUERY` on count + paginated find.

---

## GitHub comments

No open findings — prior comments resolved on branch.

---

## Findings

---
Public profile controller bypasses validatedQuery

Risk Level: HIGH  
**Status:** ✅ Fixed @ `0ac9a95`  
File Path: src/controllers/public-profile.controller.ts  
Lines: 16-17

Description:
**Contract.** `getShareView` previously parsed `req.query.page` directly despite `validate(publicProfileShareQuerySchema, "query")`.

Impact:
- Could diverge from Joi coercion/defaults on query params

Recommendation:
Use `validatedQuery` — **done**:

```typescript
const { page = 1 } = ((req as ValidatedQueryRequest).validatedQuery ??
  {}) as PublicProfileShareQuery;
```

---

---
Profile public video filter inconsistent with video share eligibility

Risk Level: HIGH  
**Status:** ✅ Fixed @ `0ac9a95`  
File Path: src/repositories/video.repository.ts  
Lines: 220-224

Description:
**Global consistency / Contract.** `publicProfileVideosFilter` previously used `VIDEO_PUBLIC_ONLY_QUERY`, allowing video-library public rows in the profile grid that 404 on `/share/video/:id`.

Impact:
- Broken profile grid links and inflated video counts

Recommendation:
Use `VIDEO_PUBLIC_SHARE_QUERY` — **done**, with `tests/repositories/video.repository.test.ts` asserting filter on count and paginated find.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Public profile controller bypasses validatedQuery | HIGH | ✅ Fixed | src/controllers/public-profile.controller.ts | 16-17 |
| 2 | Profile public video filter inconsistent with video share eligibility | HIGH | ✅ Fixed | src/repositories/video.repository.ts | 220-224 |

**Merge readiness:** **Approve for merge** — no open Critical/High findings.
