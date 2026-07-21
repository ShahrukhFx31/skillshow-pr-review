# PR review — skillshow #244 (SKSH-389)

**Repo:** SkillshowFx/skillshow  
**Branch:** SKSH-389 → main  
**Head:** `a766f025c630c578dac245516a1d1d6a3b664bb8`  
**Scope:** Free-edit window, library assignment clocks, edit-request create validation, video-library bulk upload  
**Prompt:** `pr-review/prompts/backend-system-prompt.md`

## GitHub comments

_(none — prior High is Fixed; no new Open Critical/High)_

## Findings

---
Complimentary edit requests created with default unpaid payment status

Risk Level: HIGH
File Path: src/services/edit-request.service.ts
Lines: 201-210

Description:
Previously `create` validated free-edit eligibility but left `paymentStatus` at schema default `"unpaid"`.

**Re-verify (2026-07-21):** `assertRequestedVideosEligibleForCreate` now returns `boolean`; complimentary creates set `paymentStatus: "paid"`. Covered by unit tests for in-window library delivery. Complimentary path narrowed to library deliveries only (not SkillShow uploaded), with one-per-video non-cancelled guard.

Impact:
- _(resolved)_ Admin unpaid queue vs athlete complimentary UI mismatch.

Recommendation:
- _(done)_ Keep `paymentStatus: "paid"` on complimentary create; continue asserting eligibility server-side.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Complimentary edit requests created with default unpaid payment status | HIGH | ✅ Fixed | src/services/edit-request.service.ts | 201-210 |

**Merge readiness:** No open Critical/High blockers on backend. Pair with admin-ui #350 (frontend Medium copy cleanup optional).
