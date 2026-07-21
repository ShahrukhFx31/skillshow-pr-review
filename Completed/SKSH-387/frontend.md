# PR review — skillshow-admin-ui #349 (SKSH-387)

**Repo:** SkillshowFx/skillshow-admin-ui  
**Branch:** SKSH-387 → main  
**Head:** `78463898948c899f6d5c707a29283a67908e63de`  
**Paired backend:** SkillshowFx/skillshow#245  
**Scope:** Super Admin dashboard layout, KPI order, live-ops table links  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`

## GitHub comments

_(none — prior High is Fixed via paired backend #245)_

## Findings

---
Dashboard drill-down links missing API contract fields

Risk Level: HIGH
File Path: src/pages/dashboard/superAdmin/utils/super-admin.utils.tsx
Lines: 93-137

Description:
**Contract / Global consistency.** UI links require `seq` (crew) and `slug` (events). Initially flagged because `skillshow` `main` did not return those fields.

**Re-verify (2026-07-21):** Developer confirmed fixed. Matching backend PR [skillshow#245](https://github.com/SkillshowFx/skillshow/pull/245) (`SKSH-387`) selects User `seq` and Event `slug` and maps them into `operations.activeCrew` / `upcomingEvents`. Frontend already maps `seq` and passes through event rows including `slug`, with null/empty fallbacks when absent.

Impact:
- _(resolved when #245 ships with #349)_ Dashboard drill-down links work with API payload.

Recommendation:
- _(done on #245)_ Merge frontend #349 with backend #245 so the contract lands together.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Dashboard drill-down links missing API contract fields | HIGH | ✅ Fixed | src/pages/dashboard/superAdmin/utils/super-admin.utils.tsx | 93-137 |

**Merge readiness:** No open Critical/High/Medium blockers on frontend when paired with skillshow#245.
