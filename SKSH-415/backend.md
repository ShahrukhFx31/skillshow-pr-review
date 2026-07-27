# PR review — skillshow #250 (SKSH-415)

**Repo:** SkillshowFx/skillshow  
**Branch:** SKSH-415 → main  
**Head:** `c4045b2e3ec22dff4bdfbabf2f1d5e6fd8a6e1f7`  
**Scope:** Allow coach team create with optional / empty `memberAthleteIds`  
**Prompt:** `pr-review/prompts/backend-system-prompt.md`  
**Paired frontend:** `pr-review/SKSH-415/frontend.md` (admin-ui #357)

## GitHub comments

_(none — no Open Critical/High)_

## Findings

_No Critical or High findings._

**Positive notes:** Joi `.optional().default([])`, swagger required list, and service-level empty check stay aligned. Tests cover empty create and ineligible ids filtered to `[]`. Linked-athlete eligibility still enforced for provided ids.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|

**Merge readiness:** No open Critical/High blockers on backend. Frontend #357 still has open Critical `activeTab` wiring (`"roster"` vs intentional `"athletes"` value).
