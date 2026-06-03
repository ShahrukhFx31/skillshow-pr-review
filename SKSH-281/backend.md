# Backend PR Review — skillshow (`SKSH-281`)

**Repo:** skillshow  
**Branch:** `sksh-281`  
**Base:** `main...HEAD`  
**Commits:** `6b8d42f` (fix: 281), merge from `main`  
**Scope:** Coach team list pagination/search (`GET /v1/coach/teams`) — layer separation, MongoDB performance, validation/security, types (Critical and High only)

---

## Overview

`listTeams` changes from an unbounded array to a paginated `CoachTeamsPage` (`items`, `total`, `page`, `pageSize`). The controller parses `page`, `pageSize`, and `q` via `parseCoachTeamListQuery`; the repository adds `coachTeamListFilter` (escaped case-insensitive name regex), `countTeamsByCoachList`, and skip/limit on `findTeamsByCoachListLean`; the service caps `pageSize` at 100 and maps rows through `mapCoachTeamListItem`. Swagger and unit tests are updated.

---

## Findings

No **Critical** or **High** issues identified in scope.

---

## Positive notes

- **Layering:** Controller → service → repository; mapping and pagination math stay in the service.
- **Query safety:** Search uses `escapeRegexSource` before `$regex` (same pattern as other list filters).
- **Reads:** `.lean()` + `.select()` on the list path; soft-delete still enforced via the Team schema pre-find hook (`isDeleted: false`).
- **Pagination:** `pageSize` capped at 100; invalid coach id returns an empty page with stable shape (no throw).
- **Types:** `CoachTeamsPage`, `CoachTeamsListOpts`, and `CoachTeamListItem` live in `src/types/coach-dashboard.types.ts`.
- **Tests:** Controller and service tests assert paginated response and query passthrough.

**Note (not raised as High):** Coach endpoints continue to use util query parsing on `req.query` rather than Joi + `validatedQuery`, consistent with existing `parseCoachAthleteListQuery` on sibling routes. Parallel `countDocuments` + `find` matches other coach list patterns in this module.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|

*No Critical/High findings.*

**Merge readiness:** No open backend Critical/High blockers on `sksh-281`.
