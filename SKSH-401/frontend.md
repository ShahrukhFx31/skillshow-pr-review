# PR review — skillshow-admin-ui #351 (SKSH-401)

**Repo:** SkillshowFx/skillshow-admin-ui  
**Branch:** SKSH-401 → main  
**Scope:** App user form layout, StateCityFormFields, crew breadcrumb sync, admin copy  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`

## GitHub comments

_(none — no Open Critical/High/Medium)_

## Findings

_No Critical, High, or Medium findings._

**Positive notes:** Breadcrumb parent fix in `bread-crumb.tsx` avoids linking dropdown parents incorrectly. `StateCityFormFields` `cityFirst` / `fieldCol` props are applied consistently in read-only and editable paths. Crew onboarding/work `useLayoutEffect` breadcrumb hydration matches dashboard deep links from SKSH-387.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|

**Merge readiness:** No open Critical/High/Medium blockers.
