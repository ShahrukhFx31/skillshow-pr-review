# Backend PR Review — skillshow (`SKSH-357`)

**Repo:** skillshow — `git@github-work:SkillshowFx/skillshow.git`  
**Branch:** `SKSH-357`  
**Base:** `main...HEAD` @ `24fe8cd`  
**Initial review:** 2026-06-22  
**Re-review:** 2026-06-22  
**Scope:** Multi-country state/province support (`US` / `CA` / `JP`), profile country validation, legacy US row handling, country-scoped cities endpoint (Critical / High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules / Security)

**Findings:** 1 (0 Critical, 1 High) — **0 Open**

### Protected modules

| Module | Status |
|--------|--------|
| `list-query.validation.ts`, `list-query-aggregation.utils.ts`, audit-log stack, change-stream modules | **Not modified** ✅ |

### Security scan (`SECURITY-AUDIT-PRE-RELEASE.md`)

| Check | Verdict |
|-------|---------|
| No route auth, middleware, upload, or export changes | ✅ |
| States API remains public read-only (pre-existing pattern) | ✅ |
| Profile country validation still gates writes via Joi + service normalization | ✅ |
| No weakened `authorize`, IDOR, or S3 paths touched | ✅ |

### Files reviewed

| File | Change |
|------|--------|
| `src/constants/country.constants.ts` | `STATE_COUNTRY_CODES`, `PROFILE_ACCOUNT_COUNTRIES`, legacy `USA` mapping |
| `src/controllers/state.controller.ts` | `?country=` on `getAllStates` and `getCitiesByState` + 400 on invalid |
| `src/models/state.model.ts` | `country` field; compound unique `{ country, state_id }` |
| `src/repositories/state.repository.ts` | `buildStateCountryFilter`; `$and` merge in `buildStateKeyDocumentFilter`; scoped `getCitiesByState` |
| `src/services/state.service.ts` | `countryCode` on `getAllStates` and `getCitiesByState` |
| `src/routes/state.routes.ts` | Route doc updated for cities `?country=` |
| `src/services/profile.service.ts` | Country normalize on read/write |
| `src/utils/country.utils.ts` | `parseStatesApiCountryParam`, profile country normalization |
| `src/validation/profile-account-general.validation.ts` | Canada/Japan + legacy `USA` normalization |
| Tests | Controller, repository (cross-country `ON` collision), service, utils, validation |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Country labels centralized in `country.constants.ts` | ✅ DRY |
| `parseStatesApiCountryParam` shared by list and cities controllers | ✅ Reusability |
| `buildStateKeyDocumentFilter` merges `buildStateCountryFilter` via `$and` (legacy US rows on cities path) | ✅ Global consistency |
| `getAllStates` and `getCitiesByState` both country-scoped end-to-end | ✅ |
| `resolveCanonicalStateName` still omits `countryCode` | ✅ Accepted — events/import callers; US event flows; not modified in this PR |
| `searchCities` still aggregates across all countries | ✅ Accepted — events/legacy global search |

### Re-review evidence (finding #1)

Commit `ca438f3` adds `?country=` to `getCitiesByState` controller, service, and repository. `buildStateKeyDocumentFilter` now uses `$and: [stateKeyFilter, buildStateCountryFilter(countryCode)]`. Tests cover cross-country `ON` collision and legacy US rows without `country` field.

### Positive notes

- Legacy US state documents handled consistently on list **and** cities paths.
- Repository test explicitly asserts `getCitiesByState("ON", "US")` vs `("ON", "CA")` return different cities.
- Controller tests cover invalid `country` on both list and cities endpoints.

---

## GitHub comments

_No open Critical or High findings._

**Resolved (was open):** `getCitiesByState` country scoping — fixed in `ca438f3` (`state.controller.ts` `getCitiesByState` handler, `state.repository.ts` `buildStateKeyDocumentFilter` + `getCitiesByState`).

---

## Findings

---
`getCitiesByState` not country-scoped after multi-country state model

Risk Level: HIGH  
File Path: src/controllers/state.controller.ts  
Lines: 59-83

Description:
**Global consistency** — Original review: cities endpoint lacked `?country=` while list was scoped. Re-review: `getCitiesByState` now parses `parseStatesApiCountryParam`, passes `countryCode` to service/repository, and repository merges `buildStateCountryFilter` into the lookup.

Impact:
- Was: ambiguous `findOne` when `state_id` collided across countries.
- Now: resolved; tests cover US/CA `ON` collision and legacy US rows.

Recommendation:
N/A — implemented.

**Re-review:** ✅ Fixed @ `ca438f3` / `24fe8cd`.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | `getCitiesByState` not country-scoped after multi-country state model | HIGH | ✅ Fixed | src/controllers/state.controller.ts | 59-83 |

**Merge readiness:** ✅ No Critical or High blockers on the backend. Country-scoped states and cities endpoints are aligned; legacy US rows handled on both paths; tests cover cross-country collision.
