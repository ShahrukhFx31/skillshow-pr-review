# Backend PR Review — skillshow (`SKSH-295`)

**Repo:** skillshow (main API)  
**Branch:** `SKSH-295` (merged to `main` via PR #168 @ `e1c0450`)  
**Re-verified:** `d4aa060` / `e7744b8` on branch; same code on `origin/main`  
**Scope:** Crew onboarding API, state/city search, login `isCrewOnboarded` flag (Critical & High only)  
**Findings:** 4 (0 Critical, 4 High) — **3 fixed**, **1 accepted**, **0 open**, **0 new Critical/High**

---

---
Crew onboarding PATCH allows self-deactivation via `status`

Risk Level: HIGH  
File Path: src/validation/crew-onboarding.validation.ts  
Lines: (removed)

Description:
`patchCrewOnboardingSchema` accepted `status` (`active` | `inactive`), and `CrewOnboardingService.applyUserPatch` mapped it to `user.isActive`. The route is crew-only (`authorize({ roles: [CREW_RBAC_ROLE_NAME] })`), so any crew member could PATCH `{ "status": "inactive" }` and deactivate their own account without admin involvement. The onboarding UI does not expose this field, but the API contract did.

Impact:
- Crew users can lock themselves out or hide from assignment lists while still holding credentials
- Breaks the expectation that account activation is admin-controlled (as in `crew-user.service` admin flows)

Recommendation:
Remove `status` from the crew self-service schema and `applyUserPatch`, or restrict deactivation to admin-only crew-user routes. If status is only needed for admin reads, return it in `toResponse` but do not accept it on `PATCH /v1/crew/onboarding`.

**PR comment (line 19):** **High:** Crew onboarding `PATCH` accepts `status` and writes `user.isActive`, so a crew user can deactivate themselves via API. Please drop `status` from the self-service schema/service (admin routes only).

**Re-review:** ✅ Fixed @ `e7744b8` — Still correct on `origin/main` (May 2026). No `status` in schema or `applyUserPatch`; read-only in `toResponse`.

---

---
`reportingManagerId` patch skips editor-role validation

Risk Level: HIGH  
File Path: src/services/crew-onboarding.service.ts  
Lines: 117-120

Description:
`applyCrewPatch` converted `reportingManagerId` to an `ObjectId` when non-empty but did not verify active editor role (unlike admin `crew-user` create/update).

Impact:
- Invalid reporting-manager relationships in crew records
- Assignment and reporting workflows may reference users who are not editors

Recommendation:
Reuse shared validation via `resolveCrewReportingManagerId` in `src/utils/crew.utils.ts` (also used by `crew-user.service`).

**PR comment (line 121):** **High:** Onboarding patch only checks ObjectId shape for `reportingManagerId`; admin create uses editor-role validation. Please reuse that validation here.

**Re-review:** ✅ Fixed @ `e7744b8` — Still correct on `origin/main`. `resolveCrewReportingManagerId` shared; rejection test present.

---

---
`listReportingManagers` uses unvalidated `req.query`

Risk Level: HIGH  
File Path: src/controllers/crew-onboarding.controller.ts  
Lines: 59-63

Description:
`listReportingManagers` read `req.query["search"]` directly instead of `req.validatedQuery` after a Joi query schema on the route.

Impact:
- Unbounded/untyped query input at the controller boundary
- Harder to enforce max length, coercion, and consistent API contracts

Recommendation:
Add `validate(reportingManagersQuerySchema, "query")` on the route and read `req.validatedQuery.search`.

**PR comment (line 56):** **High:** Please validate `search` with Joi on the route and use `req.validatedQuery` instead of raw `req.query` for reporting-manager lookup.

**Re-review:** ✅ Fixed @ `e7744b8` — Still correct on `origin/main`. Route + `validatedQuery` in controller.

---

---
User and crew updates in `patchForUser` are not atomic

**Status: Accepted** — team decision; rare failure path (crew save after user save). Self-service onboarding steps typically patch one layer or both in sequence; transactional follow-up deferred.

Risk Level: HIGH  
File Path: src/services/crew-onboarding.service.ts  
Lines: 30-31, 94-136

Description:
`patchForUser` still calls `applyUserPatch` (which may `user.save()`) before `applyCrewPatch`. If the crew `findOneAndUpdate` fails after the user document was saved, the request leaves split state (e.g. email/phone updated on `User` but crew fields unchanged). Editor validation now runs before crew write, but a failed crew DB update after a successful `user.save()` still splits data.

Impact:
- Partial onboarding data after transient DB errors or crew update failures
- Harder support/debugging when user and crew profiles disagree

Recommendation:
Use a MongoDB transaction/session when both layers change, or defer `user.save()` until after a successful crew update (or save both in one transaction).

**PR comment (line 30):** **High:** `patchForUser` can persist user fields before crew update completes—consider a transaction or deferred `user.save()` so user/crew stay consistent on failure.

**Re-review:** Accepted — not implemented (by design). Re-checked `origin/main` May 2026; order unchanged, no transaction.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Crew PATCH allows self-deactivation via `status` | HIGH | ✅ Fixed | `src/validation/crew-onboarding.validation.ts` | — |
| 2 | `reportingManagerId` patch skips editor-role validation | HIGH | ✅ Fixed | `src/utils/crew.utils.ts`, `src/services/crew-onboarding.service.ts` | 117-120 |
| 3 | `listReportingManagers` uses unvalidated `req.query` | HIGH | ✅ Fixed | `src/controllers/crew-onboarding.controller.ts` | 59-63 |
| 4 | User + crew patch not atomic | HIGH | Accepted | `src/services/crew-onboarding.service.ts` | 30-31, 94-136 |

**Merge status:** Backend SKSH-295 is on `main` (PR #168). No commits after `e7744b8` on the feature branch.

**Positive notes:** Shared `resolveCrewReportingManagerId`, aligned weekday labels, solid tests for reporting-manager rejection and validated query.

**New issues:** None at Critical/High.

**Skipped (per prompt):** Medium issues (e.g. `POST /complete` without required-field enforcement—intentional with “Skip Onboarding”; public `searchCities` without query Joi).
