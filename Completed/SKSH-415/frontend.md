# PR review — skillshow-admin-ui #357 (SKSH-415)

**Repo:** SkillshowFx/skillshow-admin-ui  
**Branch:** SKSH-415 → main  
**Head:** `6a2201bd00d737d0143d527536e2a15cfbfff167`  
**Scope:** Optional roster on team create; rename Athletes → Roster in UI; logo Form.Item validation  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`  
**Paired backend:** `pr-review/SKSH-415/backend.md` (skillshow #250)  
**Updated:** 2026-07-28 — findings #1 and #2 marked Accepted per reviewer

## GitHub comments

_(none — no Open findings)_

## Findings

---
activeTab checks use "roster" but Segmented value is "athletes"

Risk Level: CRITICAL  
**Status:** Accepted — deferred / intentional per reviewer (2026-07-28)  
File Path: src/pages/teams/details/index.tsx  
Lines: 49-53, 213, 250

Description:
**Contract.** `TEAM_DETAILS_TAB_OPTIONS` is `{ label: "Roster", value: "athletes" }` (author confirmed keeping `value: "athletes"` intentional). This PR still changes details comparisons from `"athletes"` → `"roster"`:

- `activeTab !== "roster"` (closes modal/filter)
- `activeTab === "roster"` (Add button + search/filter chrome)

Selecting the Roster tab sets `activeTab` to `"athletes"`, so those branches never run. The roster table still shows via the overview `else`, but without controls.

Author intent covers the **option value**, not the broken comparisons. Fix is to revert details checks to `"athletes"`.

Impact:
- Add-athlete CTA and roster search/filter never appear on the Roster tab.
- Add-athlete modal cannot stay open on that tab.

Recommendation:
```ts
if (activeTab !== "athletes") { ... }
// and
activeTab === "athletes" ? ( /* Add button / athletesTabBarExtra */ ) : null
```

Keep `TEAM_DETAILS_TAB_OPTIONS` as `{ label: "Roster", value: "athletes" }`.
---

---
Awkward “No rosters” / “Search rosters” copy

Risk Level: MEDIUM  
**Status:** Accepted — deferred / intentional per reviewer (2026-07-28)  
File Path: src/pages/teams/constant/teamEmptyStateCopy.ts  
Lines: 29

Description:
Empty title became `"No rosters on this team yet"` / `"No roster match your search"`; create placeholder `"Search rosters by name..."`; column `"Roster Name"`. Users manage athletes on a roster, not multiple “rosters.”

Impact:
- Confusing empty/search/column copy.

Recommendation:
e.g. `"No athletes on this team yet"`, `"Search roster by name..."`, column `"Name"` or `"Athlete"`. Tab label “Roster” can stay.
---

**Positive notes:** Optional roster on create + logo `Form.Item` validation align with backend #250. Create submit no longer requires ≥1 athlete.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | activeTab checks use "roster" but Segmented value is "athletes" | CRITICAL | Accepted | src/pages/teams/details/index.tsx | 49-53, 213, 250 |
| 2 | Awkward “No rosters” / “Search rosters” copy | MEDIUM | Accepted | src/pages/teams/constant/teamEmptyStateCopy.ts | 29 |
| 3 | Keep label Roster + value "athletes" | — | Accepted | src/pages/teams/constant/teamUi.constants.ts | 12 |

**Merge readiness:** No open Critical/High blockers (findings #1–#2 Accepted).
