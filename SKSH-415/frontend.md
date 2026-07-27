# PR review — skillshow-admin-ui #357 (SKSH-415)

**Repo:** SkillshowFx/skillshow-admin-ui  
**Branch:** SKSH-415 → main  
**Head:** `6a2201bd00d737d0143d527536e2a15cfbfff167`  
**Scope:** Optional roster on team create; rename Athletes → Roster in UI; logo Form.Item validation  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`  
**Paired backend:** `pr-review/SKSH-415/backend.md` (skillshow #250)  
**Updated:** 2026-07-27 — finding #1 Accepted (developer: intentional `value: "athletes"`)

## GitHub comments

### `src/pages/teams/constant/teamEmptyStateCopy.ts`

- **L29** — Awkward “No rosters” empty-state copy

## Findings

---
Tab value still "athletes" but comparisons use "roster"

Risk Level: CRITICAL
File Path: src/pages/teams/details/index.tsx
Lines: 49-53, 213, 250

Description:
**Contract.** `TEAM_DETAILS_TAB_OPTIONS` uses `value: "athletes"` while the details page compares `activeTab` to `"roster"`. Selecting the Roster tab sets `activeTab` to `"athletes"`, so roster chrome (`activeTab === "roster"`) never matches.

**Developer reply** ([discussion](https://github.com/SkillshowFx/skillshow-admin-ui/pull/357#discussion_r3655096048)): “I add it intentionaly” on keeping `value: "athletes"` in `teamUi.constants.ts`.

**Accepted:** Label “Roster” + internal value `"athletes"` is intentional per author. If details still checks `"roster"`, that remains a separate wiring bug to align to `"athletes"` — treated as Accepted for this review thread per author intent on the constant.

Impact:
- (Acknowledged) Internal tab key stays `"athletes"` by design.

Recommendation:
_(Accepted — intentional.)_ Ensure every `activeTab` check uses `"athletes"` to match the intentional option value.
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
| 1 | Tab value still "athletes" but comparisons use "roster" | CRITICAL | Accepted | src/pages/teams/details/index.tsx | 49-53, 213, 250 |
| 2 | Awkward “No rosters” / “Search rosters” copy | MEDIUM | Open | src/pages/teams/constant/teamEmptyStateCopy.ts | 29 |

**Merge readiness:** No open Critical/High blockers. Open Medium #2 (copy) only.
