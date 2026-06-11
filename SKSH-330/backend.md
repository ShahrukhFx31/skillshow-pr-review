# Backend PR Review — skillshow (`SKSH-330`)

**Repo:** skillshow (main API) — `https://github.com/fx31labs-mvp/skillshow.git`  
**Branch:** `sksh-330`  
**Base:** `main...HEAD` @ `eb7cc58d`  
**Initial review:** 2026-06-11  
**Scope:** Social platform settings model/API, global vendor availability enforcement on connect/distribute/retry (Critical / High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [frontend.md](./frontend.md), [orchestrator.md](./orchestrator.md)

**Findings:** 2 (0 Critical, 1 High, 1 Accepted) — **1 Open**

### Protected modules

| Module | Status |
|--------|--------|
| `list-query.validation.ts` | **Modified (format-only)** — whitespace collapse on `listQueryOptionalLimitField`; no behavioral change |
| `list-query-aggregation.utils.ts`, `list-row-repository.utils.ts`, `mongo-change-stream.service.ts`, `change-stream.utils.ts`, `audit-log.*` | **Not modified** |

### Files reviewed

| File | Change |
|------|--------|
| `src/base/BaseController.ts` | `unprocessableEntity` helper + 422 mapping in `respondIfAppError` |
| `src/constants/social-platform.constants.ts` | Managed vendors, sort fields, pagination bounds (max pageSize 50) |
| `src/controllers/social-platform.controller.ts` | List + PATCH status endpoints |
| `src/controllers/distribution.controller.ts` | `assertVendorsActive` on submit/retry; `respondIfAppError` on errors |
| `src/controllers/vendor.controller.ts` | `respondIfAppError` on connect flows |
| `src/errors/app-error.ts` | `UnprocessableEntityError` |
| `src/mocks/permission.seed.ts` | Social Platforms menu permission seed |
| `src/models/social-platform-setting.model.ts` | Mongo model for per-vendor active flag |
| `src/repositories/social-platform.repository.ts` | CRUD + default seeding |
| `src/repositories/vendor-log.repository.ts` | `findFailedVendorNamesByJobId` for retry-all guard |
| `src/routes/social-platform.routes.ts` | GET (authenticated) + PATCH (admin) |
| `src/services/social-platform.service.ts` | List/paginate, setActive, assertVendorsActive, filterActiveVendors |
| `src/services/vendor.service.ts` | assertVendorsActive on connect start/callback |
| `src/utils/social-platform-vendor.utils.ts` | x → twitter normalization |
| `src/validation/social-platform.validation.ts` | Query/body/param Joi schemas |
| `tests/services/social-platform.service.test.ts` | Service unit tests |
| `tests/utils/social-platform-vendor.utils.test.ts` | Vendor normalization tests |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| `assertVendorsActive` wired in vendor connect, distribution submit, retry-all, retry-vendor | ✅ Global consistency |
| `respondIfAppError` added to distribution/vendor controllers for 422 platform errors | ✅ |
| `normalizeManagedSocialVendorId` centralizes x/twitter alias (mirrors frontend util) | ✅ DRY |
| `UnprocessableEntityError` + `BaseController.unprocessableEntity` for blocked platforms | ✅ |
| Default seeding with duplicate-key race handling | ✅ |
| List controller reads `req.query` instead of `validatedQuery` | ❌ Contract — see #1 |
| Custom in-memory list schema (not `createListQuerySchema`) | ✅ Accepted — fixed 6-vendor catalog; `sortField`/`sortOrder` differ from generic list contract |
| Protected `list-query.validation.ts` touched (format-only) | ✅ Accepted — no logic change |

### Positive notes

- **Enforcement depth:** Platform availability is checked at OAuth start/callback, distribution submit, per-vendor retry, and retry-all (failed vendors only).
- **Sensible defaults:** Missing DB rows default to active; `ensureDefaults` seeds all managed vendors on first access.
- **Tests:** Good unit coverage for pagination, sort, alias normalization, assert/filter active vendors.
- **Error surface:** 422 responses use `isDisplayMessage` via `respondIfAppError`, pairing with the frontend toast path for mutation/distribution errors.

---

## GitHub comments (Open findings)

### 1. `skillshow/src/controllers/social-platform.controller.ts` line 22

**PR comment (line 22):** **High (Contract):** The list route validates query params with `validate(listSocialPlatformsQuerySchema, "query")`, which sets `req.validatedQuery`, but this controller casts `req.query` instead. Please read from `(req as ValidatedQueryRequest).validatedQuery` so Joi coercion, stripping, and defaults reach the service — same pattern as `app-user.controller.ts`, `crew-user.controller.ts`, and `event.controller.ts`.

---

## Findings

---
List controller reads req.query instead of validatedQuery

Risk Level: HIGH  
**Status:** Open  
File Path: skillshow/src/controllers/social-platform.controller.ts  
Lines: 22-26

Description:
**Contract.** `validate(listSocialPlatformsQuerySchema, "query")` runs on the list route and stores the validated payload on `req.validatedQuery` (see `validate.middleware.ts` — Express `req.query` is read-only and must not be used for typed list params). `SocialPlatformController.list` casts `req.query as SocialPlatformListQuery` instead.

```22:26:skillshow/src/controllers/social-platform.controller.ts
      const query = req.query as SocialPlatformListQuery;
      const page = await socialPlatformService.listPlatformsPaginated(
        userId,
        query
      );
```

Impact:
- Validated/stripped query values from Joi are ignored; controller relies on raw query strings.
- Type coercion and `stripUnknown` benefits are bypassed — inconsistent with `app-user`, `crew-user`, `event`, and other list controllers.
- Future schema defaults/constraints added to Joi will not reach the service unless `validatedQuery` is used.

Recommendation:
Import `ValidatedQueryRequest` from `../types/common` and replace line 22:

```typescript
const query = ((req as ValidatedQueryRequest).validatedQuery ??
  {}) as SocialPlatformListQuery;
```

**PR comment (`social-platform.controller.ts` line 22):** **High (Contract):** The list route validates query params with `validate(listSocialPlatformsQuerySchema, "query")`, which sets `req.validatedQuery`, but this controller casts `req.query` instead. Please read from `(req as ValidatedQueryRequest).validatedQuery` so Joi coercion, stripping, and defaults reach the service — same pattern as `app-user.controller.ts`, `crew-user.controller.ts`, and `event.controller.ts`.

---

---
Protected list-query.validation.ts modified (format-only)

Risk Level: HIGH  
**Status:** Accepted  
File Path: skillshow/src/validation/list-query.validation.ts  
Lines: 33-36

Description:
**Protected module.** PR reformats `listQueryOptionalLimitField` to a single line. No validation logic, defaults, or bounds changed.

Impact:
- None — cosmetic diff only.

Recommendation:
No action required. Avoid further edits to protected modules in feature tickets unless explicitly scoped.

**PR comment:** N/A — Accepted (format-only protected-module touch; no inline comment required).

---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | List controller reads req.query instead of validatedQuery | HIGH | Open | skillshow/src/controllers/social-platform.controller.ts | 22-26 | 22 |
| 2 | Protected list-query.validation.ts modified (format-only) | HIGH | Accepted | skillshow/src/validation/list-query.validation.ts | 33-36 | — |

**Merge readiness:** **Not merge-ready** — 1 open High finding (`validatedQuery` contract). Fix before merge.
