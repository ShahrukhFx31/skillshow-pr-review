# PR review — skillshow-admin-ui #361 (SKSH-429)

**Repo:** SkillshowFx/skillshow-admin-ui  
**Branch:** SKSH-429 → main  
**Head:** `9b30c3681726fac64286b1e0153d3e02852050af`  
**Scope:** Hide profile-completion for Editor (shared staff role list); shared Quick Connect copy; SkillShow branding; account GeneralTab `hasRole` + SuperAdmin on admin-like profile  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`

## GitHub comments

_(none — no Open Critical/High/Medium)_

## Findings

_No Critical, High, or Medium findings._

**Positive notes:**
- `DASHBOARD_ROLES_WITHOUT_PROFILE_COMPLETION` centralizes staff roles that skip the welcome completion ring (Admin / SuperAdmin / Editor).
- `QUICK_CONNECT_DESCRIPTION` removes duplicated Connect Platforms / Quick Connect sub-text.
- Account GeneralTab extracts `hasRole` and includes `SuperAdmin` in `isAdminLikeProfile` (previous gap).
- Share shell branding `SkillShow` is label-only.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|

**Merge readiness:** No open Critical/High/Medium blockers.
