# PR review — skillshow-admin-ui #360 (SKSH-430)

**Repo:** SkillshowFx/skillshow-admin-ui  
**Branch:** SKSH-430 → main  
**Head:** `5daab54377c667b553137286cd1ecc93a1527d98` (updated — dashboard Create Team removed)  
**Scope:** Admin teams via `/v1/admin/*` on shared Teams pages; coach filter; create-with-coach  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`  
**Paired backend:** `pr-review/SKSH-430/backend.md` (skillshow #251)  
**Updated:** 2026-07-24 — follow-up from [frontend review](d6bf1e8f-c194-4bdb-b20e-ef24c421d2a5) + re-verify on new head

## GitHub comments

### `src/pages/teams/index.tsx`

- **L51** — Coach filter/options capped at first 100 coaches

### `src/pages/teams/create/index.tsx`

- **L100** — Duplicate coach-options fetch (DRY with list page)

### `src/pages/management/app-users/teams/dashboard/index.tsx`

- **L217** — Create Team entry points removed from coach dashboard

### `src/pages/teams/details/components/AddAthletesToTeamModal.tsx`

- **L101** — Ad-hoc query-key tags instead of `adminCoachLinkedAthletes*` helpers

## Findings

---
Coach filter/options capped at first 100 coaches

Risk Level: HIGH
File Path: src/pages/teams/index.tsx
Lines: 46-55

Description:
**Contract / UX.** Admin coach filter (and create-page coach Select) load coaches via `listAppUsers({ ...DEFAULT_LIST_QUERY, pageSize: 100, role: coach })` with no search/pagination. Orgs with >100 active coaches cannot filter or assign coaches beyond the first page.

Impact:
- Missing coaches in filter/create Select.
- Prefill from management deep-links may show a bare id.

Recommendation:
Searchable async Select (debounced `q`), shared via one hook; ensure URL `coachUserId` is present in options.
---

---
Duplicate coach-options fetch across list and create

Risk Level: HIGH
File Path: src/pages/teams/create/index.tsx
Lines: 100-116

Description:
**DRY / Global consistency.** Identical `listAppUsers` + options mapping on list and create with different literal query keys (`admin-teams-coach-filter-options` vs `admin-create-team-coach-options`), not registered in `teamQueryKeys`. Mirror `useCoachTeamSelectOptions` pattern.

Impact:
- Divergent limits/labels; cache never shared; >100 coach cap duplicated.

Recommendation:
Extract `useAdminCoachSelectOptions()` + `teamQueryKeys` entry; use from list filter and create form.
---

---
Create Team entry points removed from coach dashboard

Risk Level: HIGH
File Path: src/pages/management/app-users/teams/dashboard/index.tsx
Lines: 196-218

Description:
**Global consistency.** Latest commit removes the (previously hidden) toolbar Create Team button and replaces the empty-state `DataNotFound` Create CTA with a plain `Empty` — no navigation to `/teams/create?coachUserId=...`. Admins lose the coach-scoped create entry this PR was meant to unlock from management.

Impact:
- Cannot create a team for a coach from the app-user Teams dashboard (0 or N teams).

Recommendation:
Restore Create Team (toolbar + empty state) and navigate to `/teams/create?coachUserId=${encodeURIComponent(userId)}`.
---

---
Ad-hoc query-key tags instead of adminCoachLinkedAthletes* helpers

Risk Level: HIGH
File Path: src/pages/teams/details/components/AddAthletesToTeamModal.tsx
Lines: 101-114, 128

Description:
**DRY / Global consistency.** PR adds `teamQueryKeys.adminCoachLinkedAthletes` and uses it on create, but this modal appends `isAdmin` / `coachUserId` onto coach-only `coachLinkedAthletesModal*` keys instead of dedicated admin generators.

Impact:
- Half-migrated query-key module; admin modal cache keys diverge from create page.

Recommendation:
Add `adminCoachLinkedAthletesModal` / `...SportOptions` to `queryKeys.ts` and use them here.
---

**Positive notes:** `useIsTeamsAdmin` matches backend admin/super_admin gate. API paths align with `/v1/admin/teams*`. Optional athlete assignment and coach-gated linked-athlete queries are coherent on Teams create/details.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Coach filter/options capped at first 100 coaches | HIGH | Open | src/pages/teams/index.tsx | 46-55 |
| 2 | Duplicate coach-options fetch across list and create | HIGH | Open | src/pages/teams/create/index.tsx | 100-116 |
| 3 | Create Team entry points removed from coach dashboard | HIGH | Open | src/pages/management/app-users/teams/dashboard/index.tsx | 196-218 |
| 4 | Ad-hoc query-key tags instead of adminCoachLinkedAthletes* helpers | HIGH | Open | src/pages/teams/details/components/AddAthletesToTeamModal.tsx | 101-114, 128 |

**Merge readiness:** Request changes — open High findings #1–#4.
