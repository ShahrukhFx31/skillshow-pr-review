# Backend PR Review — skillshow (`SKSH-357`)

**Repo:** skillshow — `git@github-work:SkillshowFx/skillshow.git`  
**Branch:** `SKSH-357`  
**Base:** `main...HEAD` @ `07cfae5`  
**Initial review:** 2026-06-22  
**Scope:** Multi-country state/province support (`US` / `CA` / `JP`), profile country validation expansion, legacy US row handling (Critical / High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules / Security)

**Findings:** 1 (0 Critical, 1 High) — **1 Open**

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
| `src/constants/country.constants.ts` | New — `STATE_COUNTRY_CODES`, `PROFILE_ACCOUNT_COUNTRIES`, legacy `USA` mapping |
| `src/constants/profile.constants.ts` | Re-export profile country constants from `country.constants.ts` |
| `src/controllers/state.controller.ts` | `GET /v1/states?country=US\|CA\|JP` parsing + 400 on invalid |
| `src/models/state.model.ts` | `country` field; compound unique `{ country, state_id }` |
| `src/repositories/state.repository.ts` | `buildStateCountryFilter` for legacy US rows; country-scoped `getAllStates` |
| `src/services/profile.service.ts` | Normalize country on read/write via `country.utils` |
| `src/services/state.service.ts` | Pass `countryCode` through `getAllStates` |
| `src/utils/country.utils.ts` | `parseStatesApiCountryParam`, profile country normalization |
| `src/validation/profile-account-general.validation.ts` | Accept `United States` / `Canada` / `Japan`; normalize legacy `USA` |
| Tests | Controller, model, repository, service, utils, validation coverage added |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Country labels centralized in `country.constants.ts`; profile re-exports | ✅ DRY |
| `parseStatesApiCountryParam` + `normalizeProfileCountryForRead/Write` shared across controller, validation, service | ✅ Reusability |
| `buildStateCountryFilter` handles legacy US rows missing `country` on list path | ✅ |
| `getAllStates` migrated to country scoping; `getCitiesByState` / `resolveStateByKey` (cities path) not scoped | ❌ **Global consistency** — see finding #1 |
| `buildStateKeyDocumentFilter` uses exact `country` equality when provided — would miss legacy US rows if wired without `buildStateCountryFilter` | ⚠️ Follow-up when fixing #1 |
| `searchCities` still aggregates across all countries | ✅ Accepted — events/legacy global search; not in ticket diff scope |

### Positive notes

- Legacy US state documents (no `country` field) are handled on the list endpoint via `$or` filter — matches production data reality.
- Compound unique index `{ country, state_id }` with model test for cross-country collision (`ON` in US vs CA).
- Profile validation and service both normalize `USA` → `United States` on read and write.
- Unit tests cover country param parsing, legacy US rows in repository, and expanded country validation.

---

## GitHub comments

> **Note:** `getCitiesByState` in `state.repository.ts` is unchanged in this PR, so GitHub will not anchor inline comments there. Use one of the **changed-line** anchors below.

### `src/controllers/state.controller.ts` — `getCitiesByState` JSDoc / handler (lines 55–71 in PR diff)

`GET /v1/states` now accepts `?country=`, but `getCitiesByState` still calls `stateService.getCitiesByState(stateId)` with no country. The JSDoc was updated for multi-country codes (`ON`, `13`), yet the handler does not parse or pass `country`. With the new compound unique `{ country, state_id }` index, the same `state_id` can exist in multiple countries — `findOne` in the repository returns an arbitrary match. Please add `?country=US|CA|JP` here (default `US`, same `parseStatesApiCountryParam` as `getAllStates`), thread through service → `stateRepository.getCitiesByState(stateKey, countryCode)`, and merge `buildStateCountryFilter` into the lookup.

**Alternate anchor (repository helper change):** `src/repositories/state.repository.ts` lines **41–43** — `countryCode` was added to `buildStateKeyDocumentFilter`, but `getCitiesByState` (unchanged below) still calls `buildStateKeyDocumentFilter(stateKey)` without it.

---

## Findings

---
`getCitiesByState` not country-scoped after multi-country state model

Risk Level: HIGH  
File Path: src/controllers/state.controller.ts (PR comment anchor); fix in `state.repository.ts` `getCitiesByState`  
Lines: 55-71 (PR diff — `getCitiesByState` handler); 160-161 (post-merge repository)

Description:
**Global consistency** — The PR adds a `country` field and compound unique index `{ country, state_id }`, and scopes `getAllStates(countryCode)` correctly (including legacy US rows via `buildStateCountryFilter`). However, `getCitiesByState` still resolves states with `buildStateKeyDocumentFilter(stateKey)` and no country dimension. The controller (`GET /v1/states/:stateId/cities`) and service layer were not updated to accept `?country=`. When `state_id` values collide across countries (explicitly allowed by the new model test), `StateModel.findOne` returns an undefined match — wrong city list for CA/JP profile users.

Impact:
- Canada/Japan users can see incorrect cities after selecting a province/prefecture when `state_id` collides with another country’s row.
- Frontend `useStateAndCitySelector` fetches states with `countryCode` but cities without it, so the list and cities endpoints are inconsistent.
- Undermines the purpose of the multi-country states feature on the primary profile edit flow.

Recommendation:
1. Add optional `?country=US|CA|JP` to `GET /v1/states/:stateId/cities` (default `US`); validate with `parseStatesApiCountryParam` in `state.controller.ts`.
2. Extend `stateService.getCitiesByState(stateKey, countryCode)` and `stateRepository.getCitiesByState(stateKey, countryCode)`.
3. Replace the bare `filter["country"] = countryCode` pattern in `buildStateKeyDocumentFilter` with merging `buildStateCountryFilter(countryCode)` when scoping lookups (so legacy US rows without `country` still resolve).
4. Add repository test: seed `ON` in US and CA, assert `getCitiesByState("ON", "CA")` returns only Canadian cities.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | `getCitiesByState` not country-scoped after multi-country state model | HIGH | Open | src/controllers/state.controller.ts | 55-71 (PR diff) |

**Merge readiness:** ❌ One open High blocker — cities endpoint must be country-scoped to match `getAllStates` and the new compound unique index before CA/JP profile flows are safe in production.
