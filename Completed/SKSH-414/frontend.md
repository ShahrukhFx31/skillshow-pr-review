# PR review — skillshow-admin-ui #356 (SKSH-414)

**Repo:** SkillshowFx/skillshow-admin-ui  
**Branch:** SKSH-414 → main  
**Head:** `5b6e69cf18ffaa39597ce52b2e26c856dae727b8`  
**Scope:** Split athlete goal options from version/output-type options  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`  
**Re-verify:** 2026-07-22 — finding #1 Accepted (intentional two-const split; VALUES scoped to athlete goals)

## GitHub comments

_(none — no Open Critical/High/Medium)_

## Findings

---
EDIT_REQUEST_GOAL_VALUES omits version/output goal slugs

Risk Level: HIGH
File Path: src/constants/edit-request-goals.constants.ts
Lines: 52-63

Description:
**DRY / Contract.** Version goals move into `EDIT_REQUEST_GOAL_OPTIONS_WITH_VERSIONS` while `EDIT_REQUEST_GOAL_VALUES` / labels stay derived from athlete `EDIT_REQUEST_GOAL_OPTIONS` only. Author confirmed the two option consts are intentional.

Impact:
- Version/output slugs may not pass `isEditRequestGoal` / `normalizeEditRequestGoals` the same way as athlete goals.

Recommendation:
N/A — Accepted as intentional differentiation between athlete goal options and version/output-type options.

Status note: **Accepted** — product/author intent is a deliberate split; no change required for merge.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | EDIT_REQUEST_GOAL_VALUES omits version/output goal slugs | HIGH | Accepted | src/constants/edit-request-goals.constants.ts | 52-63 |

**Merge readiness:** No open Critical/High/Medium blockers.
