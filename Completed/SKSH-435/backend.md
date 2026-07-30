# Backend PR Review — skillshow (`SKSH-435`)

**Repo:** SkillshowFx/skillshow  
**PR:** https://github.com/SkillshowFx/skillshow/pull/253  
**Branch:** `SKSH-435`  
**Base:** `main`  
**Head:** `fd406ae3a00957ea318865ca96380d359e9cf91b`  
**Scope:** Allow one complimentary SkillShow/library delivery mixed with owned athlete uploads on edit-request create  
**Prompt:** `pr-review/prompts/backend-system-prompt.md`  
**Paired frontend:** `pr-review/SKSH-435/frontend.md` (admin-ui #364)  
**Updated:** 2026-07-30 — re-review on latest head (#1 fixed)

## GitHub comments

_(No open inline comments — prior finding resolved.)_

## Findings

---
Mixed complimentary + athlete uploads marks whole request paymentStatus paid

Risk Level: HIGH  
File Path: src/services/edit-request.service.ts  
Lines: 211-214, 290-314

Description:
**Contract.** Earlier head returned `true` from `assertRequestedVideosEligibleForCreate` whenever a complimentary delivery was present, causing `create` to set `paymentStatus: "paid"` for mixed carts. Latest head returns `athleteUploadRows.length === 0` only for fully complimentary creates; mixed carts stay on schema default (`unpaid`).

Impact:
- Resolved — mixed delivery + athlete uploads no longer force whole-request `paid`.

Recommendation:
N/A — fixed. Test `allows mixing one free-edit delivery with athlete uploads (stays unpaid)` asserts `paymentStatus: "paid"` is not set.
---

**Positive notes:** Ownership checks for athlete extras in mixed carts; single complimentary delivery enforced; test coverage for mixed unpaid and multi-delivery rejection.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Mixed complimentary + athlete uploads marks whole request paymentStatus paid | HIGH | ✅ Fixed | src/services/edit-request.service.ts | 211-214, 290-314 |

**Merge readiness:** No open Critical/High blockers.
