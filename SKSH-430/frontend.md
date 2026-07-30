# PR review — skillshow-admin-ui #360 (SKSH-430)

**Repo:** SkillshowFx/skillshow-admin-ui  
**Branch:** SKSH-430 → main  
**Head:** `ccb38ffdd3a6edd326dd8f1432c8b3d58aa9e7ea`  
**Scope:** Admin teams via `/v1/admin/*` on shared Teams pages; coach filter; create-with-coach  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`  
**Paired backend:** `pr-review/SKSH-430/backend.md` (skillshow #251)  
**Updated:** 2026-07-30 — re-review on latest head (#3 fixed; #1–#2, #4 still open)

## GitHub comments

### `src/pages/teams/index.tsx`

- **L70** — Coach filter/options capped at first 100 coaches

### `src/pages/teams/create/index.tsx`

- **L109** — Duplicate coach-options fetch (DRY with list + edit pages)

### `src/pages/teams/details/components/AddAthletesToTeamModal.tsx`

- **L103** — Ad-hoc query-key tags instead of `adminCoachLinkedAthletes*` helpers

## Findings

---
Coach filter/options capped at first 100 coaches

Risk Level: HIGH
File Path: src/pages/teams/index.tsx
Lines: 65-74

Description:
**Contract / UX.** Admin coach filter (and create/edit coach Select) load coaches via `listAppUsers({ ...DEFAULT_LIST_QUERY, pageSize: 100, role: coach })` with no search/pagination. Orgs with >100 active coaches cannot filter or assign coaches beyond the first page.

Impact:
- Missing coaches in filter/create/edit Select.
- Prefill from management deep-links may show a bare id when coach is outside the first page (edit page partially mitigates with current-coach fallback).

Recommendation:
Searchable async Select (debounced `q`), shared via one hook; ensure URL `coachUserId` is present in options.
---

---
Duplicate coach-options fetch across list, create, and edit

Risk Level: HIGH
File Path: src/pages/teams/create/index.tsx
Lines: 106-116

Description:
**DRY / Global consistency.** Identical `listAppUsers` + options mapping appears in three places with different literal query keys (`admin-teams-coach-filter-options`, `admin-create-team-coach-options`, `admin-edit-team-coach-options` in `TeamOverviewSection`), not registered in `teamQueryKeys`. Mirror `useCoachTeamSelectOptions` pattern.

Impact:
- Divergent limits/labels; cache never shared; >100 coach cap duplicated three times.

Recommendation:
Extract `useAdminCoachSelectOptions()` + `teamQueryKeys` entry; use from list filter, create form, and team edit overview.
---

---
Create Team entry points removed from coach dashboard

Risk Level: HIGH
File Path: src/pages/management/app-users/teams/dashboard/index.tsx
Lines: 169-171, 819-827, 885-887

Description:
**Global consistency.** Earlier commit removed Create Team CTAs; latest head restores navigation via `goToCreateTeam` → `/teams/create?coachUserId=...` in the page header (when teams exist) and empty state.

Impact:
- Resolved — admins can again create a team for a coach from the app-user Teams dashboard.

Recommendation:
N/A — fixed on latest head.
---

---
Ad-hoc query-key tags instead of adminCoachLinkedAthletes* helpers

Risk Level: HIGH
File Path: src/pages/teams/details/components/AddAthletesToTeamModal.tsx
Lines: 103-116, 130

Description:
**DRY / Global consistency.** PR adds `teamQueryKeys.adminCoachLinkedAthletes` and uses it on create, but this modal appends `isAdmin` / `coachUserId` onto coach-only `coachLinkedAthletesModal*` keys instead of dedicated admin generators.

Impact:
- Half-migrated query-key module; admin modal cache keys diverge from create page.

Recommendation:
Add `adminCoachLinkedAthletesModal` / `...SportOptions` to `queryKeys.ts` and use them here.
---

**Positive notes:** `useIsTeamsAdmin` matches backend admin/super_admin gate. API paths align with `/v1/admin/teams*`. Coach dashboard deep-link create flow and post-create success modal are coherent. Optional athlete assignment and coach-gated linked-athlete queries work on Teams create/details.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Coach filter/options capped at first 100 coaches | HIGH | Open | src/pages/teams/index.tsx | 65-74 |
| 2 | Duplicate coach-options fetch across list, create, and edit | HIGH | Open | src/pages/teams/create/index.tsx | 106-116 |
| 3 | Create Team entry points removed from coach dashboard | HIGH | ✅ Fixed | src/pages/management/app-users/teams/dashboard/index.tsx | 169-171, 819-827, 885-887 |
| 4 | Ad-hoc query-key tags instead of adminCoachLinkedAthletes* helpers | HIGH | Open | src/pages/teams/details/components/AddAthletesToTeamModal.tsx | 103-116, 130 |

**Merge readiness:** Request changes — open High findings #1, #2, #4.
