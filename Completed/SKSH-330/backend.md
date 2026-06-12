# Backend PR Review — skillshow (`SKSH-330`)

**Repo:** skillshow (main API) — `https://github.com/fx31labs-mvp/skillshow.git`  
**Branch:** `sksh-330`  
**Base:** `main...HEAD` @ `66eb7230`  
**Initial review:** 2026-06-11  
**Re-reviewed:** 2026-06-11 (`66eb7230`)  
**Scope:** Social platform settings model/API, global vendor availability enforcement (Critical / High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [frontend.md](./frontend.md), [orchestrator.md](./orchestrator.md)

**Findings:** 2 (0 Critical, 1 High, 1 Accepted) — **0 Open**, **1 Fixed**, **1 Accepted**

### Protected modules

| Module | Status |
|--------|--------|
| `list-query.validation.ts` | **Modified (format-only)** — whitespace collapse; no behavioral change |
| `list-query-aggregation.utils.ts`, audit-log stack, change-stream modules | **Not modified** |

### Files reviewed

| File | Change |
|------|--------|
| `src/controllers/social-platform.controller.ts` | List + PATCH status |
| `src/services/social-platform.service.ts` | List/paginate, setActive, assertVendorsActive |
| `src/controllers/distribution.controller.ts` | assertVendorsActive on submit/retry |
| `src/services/vendor.service.ts` | assertVendorsActive on connect |
| `src/routes/social-platform.routes.ts` | GET + PATCH routes |
| `src/models/social-platform-setting.model.ts` | Mongo model |
| `src/repositories/social-platform.repository.ts` | CRUD + seeding |
| `tests/services/social-platform.service.test.ts` | Unit tests |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| `assertVendorsActive` on connect, distribute, retry paths | ✅ |
| `respondIfAppError` for 422 platform errors | ✅ |
| `normalizeManagedSocialVendorId` centralizes x/twitter alias | ✅ DRY |
| List controller uses `validatedQuery` | ✅ Fixed |
| Custom in-memory list schema (not `createListQuerySchema`) | ✅ Accepted — fixed 6-vendor catalog |
| Protected `list-query.validation.ts` format-only touch | ✅ Accepted |

### Positive notes

- **Re-review:** `validatedQuery` contract fixed on list endpoint.
- **Enforcement depth:** OAuth, distribution submit, and retry paths all guard inactive platforms.
- **Tests:** Solid unit coverage for pagination, sort, alias normalization, assert/filter.

---

## GitHub comments (Open findings)

No open findings — prior comment resolved on branch.

---

## Findings

---
List controller reads req.query instead of validatedQuery

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: skillshow/src/controllers/social-platform.controller.ts  
Lines: 25-30

Description:
**Contract.** Initial diff cast `req.query` after Joi validation on the list route.

**Re-verification (`66eb7230`):** ✅ **Fixed:**

```25:30:skillshow/src/controllers/social-platform.controller.ts
      const query = ((req as ValidatedQueryRequest).validatedQuery ??
        {}) as SocialPlatformListQuery;
      const page = await socialPlatformService.listPlatformsPaginated(
        userId,
        query
      );
```

**PR comment (`social-platform.controller.ts` line 25):** **Resolved** — reads `validatedQuery` per validate middleware contract.

---

---
Protected list-query.validation.ts modified (format-only)

Risk Level: HIGH  
**Status:** Accepted  
File Path: skillshow/src/validation/list-query.validation.ts  
Lines: 33-36

Description:
**Protected module.** Whitespace-only reformat of `listQueryOptionalLimitField`. No logic change.

**PR comment:** N/A — Accepted (no inline comment required).

---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | List controller reads req.query instead of validatedQuery | HIGH | ✅ Fixed | skillshow/src/controllers/social-platform.controller.ts | 25-30 | 25 |
| 2 | Protected list-query.validation.ts modified (format-only) | HIGH | Accepted | skillshow/src/validation/list-query.validation.ts | 33-36 | — |

**Merge readiness:** No open Critical/High blockers. Safe to merge from a backend perspective.
