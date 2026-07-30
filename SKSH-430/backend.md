# PR review — skillshow #251 (SKSH-430)

**Repo:** SkillshowFx/skillshow  
**Branch:** SKSH-430 → main  
**Head:** `0dc2337464f7c55982645cbc0b79f9b7cf946c48`  
**Scope:** Admin teams CRUD/list under `/v1/admin/teams*`, reuse coach-link service, coach reassignment + notify  
**Prompt:** `pr-review/prompts/backend-system-prompt.md`  
**Paired frontend:** `pr-review/SKSH-430/frontend.md` (admin-ui #360)  
**Updated:** 2026-07-30 — re-review on latest head (prior findings still open)

## GitHub comments

### `src/controllers/admin-teams.controller.ts`

- **L17** — List handlers read `req.query` instead of `validatedQuery`

### `src/routes/admin-teams.routes.ts`

- **L40** — Team/coach athlete list routes lack query validation

### `src/repositories/coach-link.repository.ts`

- **L384** — Admin list filter/count/find duplicate coach-scoped methods
- **L416** — `countAllTeamsList` bypasses soft-delete filter

## Findings

---
List handlers read req.query instead of validatedQuery

Risk Level: HIGH
File Path: src/controllers/admin-teams.controller.ts
Lines: 17-19, 198-200, 278-280

Description:
**Contract.** `GET /teams` wires `validate(adminTeamsListQuerySchema, "query")`, which sets `req.validatedQuery` with coerced `page` / `pageSize` (and strips unknowns). Controllers still call `parseAdminTeamListQuery` / `parseCoachAthleteListQuery` on raw `req.query`. Middleware docs require typed list params from `validatedQuery` only. Same pattern on `listTeamAthletes` and `listCoachLinkedAthletes`.

Impact:
- Joi coercion/defaults on `validatedQuery` are ignored; parsers re-interpret strings from Express query.
- Future schema fields (defaults, stricter enums) will silently diverge from what the controller uses.

Recommendation:
Pass `(req as ValidatedQueryRequest).validatedQuery ?? {}` into the parse helpers (or map `validatedQuery` directly).
---

---
Team/coach athlete list routes lack query validation

Risk Level: HIGH
File Path: src/routes/admin-teams.routes.ts
Lines: 40-44, 91-95

Description:
**Contract.** `GET /teams/:teamId/athletes` and `GET /coaches/:coachUserId/athletes` validate params only — no Joi query schema — then parse `page` / `pageSize` / sort from raw query.

Impact:
- Invalid query shapes are not rejected at the edge with consistent 400 messages.
- Bounds/allow-lists live only in ad-hoc parsers + service clamps.

Recommendation:
Add a shared coach-athlete list query schema and `validate(..., "query")`; read `validatedQuery` in the controller.
---

---
Admin list filter/count/find duplicate coach-scoped methods

Risk Level: HIGH
File Path: src/repositories/coach-link.repository.ts
Lines: 384-436, 446-454

Description:
**DRY.** New `adminTeamListFilter` / `countAllTeamsList` / `findAllTeamsListLean` nearly copy `coachTeamListFilter` / `countTeamsByCoachList` / `findTeamsByCoachListLean` (same regex search, sport, season/year parsing, skip/limit/sort). Only difference: optional vs required `coachUserId`. Also adds unused `findTeamListRowByIdLean` (no production callers).

Impact:
- Parallel list implementations will drift (filters, defaults, soft-delete).
- Dead method adds maintenance surface.

Recommendation:
Parameterize the existing coach list helpers with optional `coachUserId` and reuse them for admin. Remove `findTeamListRowByIdLean` unless a caller is added in this PR.
---

---
countAllTeamsList bypasses soft-delete filter

Risk Level: HIGH
File Path: src/repositories/coach-link.repository.ts
Lines: 410-416

Description:
**Contract.** `TeamSchema.pre(/^find/)` adds `isDeleted: false` for `find*`, but not for `countDocuments`. `findAllTeamsListLean` excludes soft-deleted rows; `countAllTeamsList` does not, so `total` can exceed returnable rows. Pre-existing `countTeamsByCoachList` has the same gap; this PR copies it into the new admin path.

Impact:
- Inflated admin Teams pagination totals / empty trailing pages after soft-deletes.

Recommendation:
Put `isDeleted: false` in the shared list filter used by both `find` and `countDocuments` (fix coach + admin counts together).
---

**Positive notes:** `authorize({ roles: ["admin"] })` expands to `super_admin`. Admin mutations resolve the team’s coach and reuse coach-link eligibility. Coach role checked on create/reassign; roster trimmed on reassignment; search uses `escapeRegexSource`. `createdBy` tracking, team-assigned notifications, and optional athlete assignment on create are well covered by new tests.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | List handlers read req.query instead of validatedQuery | HIGH | Open | src/controllers/admin-teams.controller.ts | 17-19, 198-200, 278-280 |
| 2 | Team/coach athlete list routes lack query validation | HIGH | Open | src/routes/admin-teams.routes.ts | 40-44, 91-95 |
| 3 | Admin list filter/count/find duplicate coach-scoped methods | HIGH | Open | src/repositories/coach-link.repository.ts | 384-436, 446-454 |
| 4 | countAllTeamsList bypasses soft-delete filter | HIGH | Open | src/repositories/coach-link.repository.ts | 410-416 |

**Merge readiness:** Request changes — open High findings #1–#4.
