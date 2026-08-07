# PR review — skillshow-admin-ui #360 (SKSH-430)

**Repo:** SkillshowFx/skillshow-admin-ui  
**Branch:** SKSH-430 → main  
**Head:** `e01e0849eb776a9d46c845eea8e655275149328c`  
**Scope:** Admin teams via `/v1/admin/*` on shared Teams pages; coach filter; create-with-coach  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`  
**Paired backend:** `pr-review/SKSH-430/backend.md` (skillshow #251)  
**Updated:** 2026-08-04 — re-verify on latest head (all prior High findings fixed)

## GitHub comments

_(none — no Open Critical/High/Medium findings)_

## Findings

---
Coach filter/options capped at first 100 coaches

Risk Level: HIGH
File Path: src/pages/teams/index.tsx
Lines: 64-67

Description:
**Contract / UX.** Shared `useAdminCoachSelectOptions` provides debounced server search (`listAppUsers` + `search`), ensures selected/URL `coachUserId` is always in options, and registers keys in `teamQueryKeys`.

Impact:
- Resolved — coaches beyond the first page are reachable via search; deep-link coach ids stay labeled.

Recommendation:
N/A — fixed on head `e01e0849`.
---

---
Duplicate coach-options fetch across list, create, and edit

Risk Level: HIGH
File Path: src/pages/teams/hooks/useAdminCoachSelectOptions.ts
Lines: 1-112

Description:
**DRY / Global consistency.** List filter, create form, and `TeamOverviewSection` all consume `useAdminCoachSelectOptions` with shared `teamQueryKeys.adminCoachSelectOptions` / `adminCoachSelectOptionById`.

Impact:
- Resolved — one hook, shared cache, consistent search behavior.

Recommendation:
N/A — fixed on head `e01e0849`.
---

---
Create Team entry points removed from coach dashboard

Risk Level: HIGH
File Path: src/pages/management/app-users/teams/dashboard/index.tsx
Lines: 176, 214, 263-289

Description:
**Global consistency.** Create Team CTAs route via `goToCreateTeam` → `/teams/create?coachUserId=...`.

Impact:
- Resolved — admins can create a team for a coach from the app-user Teams dashboard.

Recommendation:
N/A — fixed on prior head; unchanged.
---

---
Ad-hoc query-key tags instead of adminCoachLinkedAthletes* helpers

Risk Level: HIGH
File Path: src/pages/teams/details/components/AddAthletesToTeamModal.tsx
Lines: 103-139

Description:
**DRY / Global consistency.** Modal uses `teamQueryKeys.adminCoachLinkedAthletesModal` and `adminCoachLinkedAthletesModalSportOptions` for admin paths.

Impact:
- Resolved — query-key module fully migrated for admin modal flows.

Recommendation:
N/A — fixed on head `e01e0849`.
---

**Positive notes:** `useIsTeamsAdmin` matches backend admin/super_admin gate. API paths align with `/v1/admin/teams*`. Coach dashboard deep-link create flow and post-create success modal are coherent. Admin team table sort fields align with backend allow-list.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Coach filter/options capped at first 100 coaches | HIGH | ✅ Fixed | src/pages/teams/index.tsx | 64-67 |
| 2 | Duplicate coach-options fetch across list, create, and edit | HIGH | ✅ Fixed | src/pages/teams/hooks/useAdminCoachSelectOptions.ts | 1-112 |
| 3 | Create Team entry points removed from coach dashboard | HIGH | ✅ Fixed | src/pages/management/app-users/teams/dashboard/index.tsx | 176, 214, 263-289 |
| 4 | Ad-hoc query-key tags instead of adminCoachLinkedAthletes* helpers | HIGH | ✅ Fixed | src/pages/teams/details/components/AddAthletesToTeamModal.tsx | 103-139 |

**Merge readiness:** **Merge-ready** — ship with backend #251.
