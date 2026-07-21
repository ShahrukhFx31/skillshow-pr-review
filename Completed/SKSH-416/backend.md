# PR review — skillshow #247 (SKSH-416)

**Repo:** SkillshowFx/skillshow  
**Branch:** SKSH-416 → main  
**Head:** `5c423eaeb60623f890ed1881b2b5e43c35896b5f`  
**Scope:** Event display `location` built from street address (not facility); always include country  
**Prompt:** `pr-review/prompts/backend-system-prompt.md`

## GitHub comments

_(none — no Open Critical/High)_

## Findings

_No Critical or High findings._

**Positive notes:** `normalizeEventBody` and `toEventDto` both use `streetAddress` for the composed `location` string; facility remains a separate field. Tests and swagger example updated to match. No auth/RBAC surface change.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|

**Merge readiness:** No open Critical/High blockers.
