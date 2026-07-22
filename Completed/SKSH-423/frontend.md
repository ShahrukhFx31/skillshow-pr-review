# PR review — skillshow-admin-ui #354 (SKSH-423)

**Repo:** SkillshowFx/skillshow-admin-ui  
**Branch:** SKSH-423 → main  
**Head:** `96a8776225f202173d383f85e3e6208f40b19de7`  
**Scope:** Remove `partnerSource` from event types, forms, tables, and crew work views; alphabetize related select options  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`  
**Paired backend:** `pr-review/SKSH-423/backend.md` (skillshow #248)

## GitHub comments

_(none — no Open Critical/High/Medium)_

## Findings

_No Critical, High, or Medium findings._

**Positive notes:** Removal is applied consistently across dashboard columns, responsive tables, onboarding form/view, create/update/bulk patch helpers, list mapping, and crew-user work sort allow-list — no mixed old/new `partnerSource` usage in the PR. Sort types and `EVENT_COLUMN_KEYS` stay aligned with the backend allow-list change in #248.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|

**Merge readiness:** No open Critical/High/Medium blockers. Ship with backend #248.
