# Backend PR Review — skillshow (`DEEP-TEST`)

**Repo:** SkillshowFx/skillshow  
**PR:** https://github.com/SkillshowFx/skillshow/pull/265  
**Branch:** `deep/test` → main  
**Head:** `e9a757cdc43bde1107d9e5c2d6c57e1b27799aa2`  
**Scope:** Free-edit reopen + round output caps, partner sortOrder/contacts, social-platform list order, crew profile detail fields  
**Prompt:** `pr-review/prompts/backend-system-prompt.md`  
**Paired:** `pr-review/Completed/DEEP-TEST-265-378/frontend.md` (#378)  
**Updated:** 2026-08-20 — finding #1 fixed; archived as `Completed/DEEP-TEST-265-378`

## GitHub comments

_(none — no Open Critical/High findings)_

## Findings

---
Empty prior-round output shells inflate current free-edit output cap

Risk Level: HIGH
File Path: src/services/edit-request-output.service.ts
Lines: 541-551

Description:
**Global consistency.** `countOutputsTowardCurrentRoundCap` now uses `emptyOutputBelongsToCurrentFreeEditRound` (created-at vs latest `FREE_EDIT_REOPENED` history). Covered by `edit-request-free-edit-reopen.utils.test.ts`.

Impact:
- Resolved — prior-round empty shells no longer fill the active round cap.

Recommendation:
N/A — fixed on head `e9a757cd`.
---

**Positive notes:** Shared reopen helper keeps FE/BE empty-shell ownership aligned. Reopen stays owner-scoped. Partner audit still goes through `auditLogService`.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Empty prior-round output shells inflate current free-edit output cap | HIGH | ✅ Fixed | src/services/edit-request-output.service.ts | 541-551 |

**Merge readiness:** **Merge-ready** — all findings Fixed. Paired with frontend #378 (Medium #4 Accepted).
