# PR review — skillshow-admin-ui #357 (SKSH-415)

**Repo:** SkillshowFx/skillshow-admin-ui  
**Branch:** SKSH-415 → main  
**Head:** `fd6664dea91f5cd4aea46c59919d47d7e8aa2e96`  
**Scope:** Optional roster on team create; rename Athletes → Roster in UI; logo Form.Item validation  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`  
**Paired backend:** `pr-review/SKSH-415/backend.md` (skillshow #250)

## GitHub comments

### `src/pages/teams/details/index.tsx`

- **L49** — Tab value still `"athletes"` but comparisons use `"roster"`

### `src/pages/teams/constant/teamEmptyStateCopy.ts`

- **L29** — Awkward “No rosters” empty-state copy

## Findings

---
Tab value still "athletes" but comparisons use "roster"

Risk Level: CRITICAL
File Path: src/pages/teams/details/index.tsx
Lines: 49-53, 213, 250

Description:
**Contract.** `TEAM_DETAILS_TAB_OPTIONS` still uses `value: "athletes"` while the details page compares `activeTab` to `"roster"`. Selecting the Roster tab sets `activeTab` to `"athletes"`, so:
1. `useEffect` treats the roster tab as “not roster” and immediately closes the add-athlete modal / grad filter.
2. Card extra “Add Athlete to Team” and `athletesTabBarExtra` (search/filter) never render (`activeTab === "roster"` is always false).

`TeamAthletesSection` still appears via the overview `else` branch, so the table shows without its controls — broken roster management UX.

Impact:
- Coaches cannot reliably add athletes or filter/search the roster on team details.
- Add-athlete modal cannot stay open on the Roster tab.

Recommendation:
Keep one source of truth. Prefer renaming the option value:

```ts
// teamUi.constants.ts
{ label: "Roster", value: "roster" },
```

Or revert details comparisons to `"athletes"`. If the value changes, update every `activeTab === ...` check and any persisted tab state in the same PR.
---

---
Awkward “No rosters” / “Search rosters” copy

Risk Level: MEDIUM
File Path: src/pages/teams/constant/teamEmptyStateCopy.ts
Lines: 29

Description:
Empty title became `"No rosters on this team yet"` and create search placeholder `"Search rosters by name..."`. Users search/add athletes (roster members), not “rosters.” Column “Roster Name” is similarly unclear vs “Athlete Name” / “Name.”

Impact:
- Confusing empty/search copy on create and details.

Recommendation:
Use athlete/roster-member language, e.g. `"No athletes on this team yet"`, `"Search roster by name..."`, column title `"Name"` or `"Athlete"`. Keep the tab label “Roster” if desired.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Tab value still "athletes" but comparisons use "roster" | CRITICAL | Open | src/pages/teams/details/index.tsx | 49-53, 213, 250 |
| 2 | Awkward “No rosters” / “Search rosters” copy | MEDIUM | Open | src/pages/teams/constant/teamEmptyStateCopy.ts | 29 |

**Merge readiness:** Request changes — open Critical finding #1.
