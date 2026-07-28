# Backend PR Review — skillshow (`SKSH-435`)

**Repo:** SkillshowFx/skillshow  
**PR:** https://github.com/SkillshowFx/skillshow/pull/253  
**Branch:** `SKSH-435`  
**Base:** `main`  
**Head:** `d2e7c8556a19e77b3db6ff3d5da2ceba3e93a99f`  
**Scope:** Allow one complimentary SkillShow/library delivery mixed with owned athlete uploads on edit-request create  
**Prompt:** `pr-review/prompts/backend-system-prompt.md`  
**Paired frontend:** `pr-review/SKSH-435/frontend.md` (admin-ui #364)

## GitHub comments

### `src/services/edit-request.service.ts`

- **L311** — Mixed complimentary + athlete uploads marks whole request `paymentStatus: "paid"` (blast radius: `create()` ~213)

## Findings

---
Mixed complimentary + athlete uploads marks whole request paymentStatus paid

Risk Level: HIGH  
**Status:** Open  
File Path: src/services/edit-request.service.ts  
Lines: 200-215, 256-284

Description:
**Contract.** `assertRequestedVideosEligibleForCreate` now returns `true` whenever the cart includes one in-window library delivery (even with additional owned athlete uploads). `create` then spreads `paymentStatus: "paid"` for any complimentary path. Athlete-upload-only creates leave the schema default (`unpaid`). The paired admin-ui (#364) prices extras as `paidVideoCount * EDIT_REQUEST_BASE_PRICE_USD` and labels the cart “Complimentary delivery + paid extras,” so backend treats mixed carts as fully settled while UI/admin unpaid tracking for paid-only carts does not.

Impact:
- Mixed carts never land as `unpaid` for the paid extras portion.
- Admin payment filters / athlete payment UX diverge from the price shown on the enhance screen.
- Athlete-only vs mixed carts are inconsistent for the same paid upload rows.

Recommendation:
Only set complimentary `paymentStatus: "paid"` when the request is **fully** complimentary (`deliveryRows.length === 1 && athleteUploadRows.length === 0`), or introduce an explicit mixed payment model. Align with frontend: if extras are paid, keep default/`unpaid` (or a partial status) when `athleteUploadRows.length > 0`. Add a create test asserting mixed delivery + upload does **not** force `paymentStatus: "paid"` unless product intends free extras.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Mixed complimentary + athlete uploads marks whole request paymentStatus paid | HIGH | Open | src/services/edit-request.service.ts | 200-215, 256-284 |

**Merge readiness:** Request changes — 1 open High finding (paymentStatus contract vs paid extras).
