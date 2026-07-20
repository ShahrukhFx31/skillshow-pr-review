# PR review — skillshow #244 (SKSH-389)

**Repo:** SkillshowFx/skillshow  
**Branch:** SKSH-389 → main  
**Scope:** Free-edit window, library assignment clocks, edit-request create validation  
**Prompt:** `pr-review/prompts/backend-system-prompt.md`

## GitHub comments

### `src/services/edit-request.service.ts` (line 201)

**HIGH** — Complimentary creates still default to unpaid

## Findings

---
Complimentary edit requests created with default unpaid payment status

Risk Level: HIGH
File Path: src/services/edit-request.service.ts
Lines: 201-210

Description:
`create` validates free-edit eligibility via `assertRequestedVideosEligibleForCreate` but still calls `editRequestRepository.create` without setting `paymentStatus`. The model default remains `"unpaid"`. The admin UI treats payment status as operational signal (“Edit Queue” / payment filters). Athletes see “Complimentary” / “Free” on the frontend (#350) while the record stays unpaid unless an admin manually marks it paid.

Impact:
- Complimentary requests may sit in unpaid queues and confuse editors/admins.
- Risk of payment follow-up for requests that should bypass billing.

Recommendation:
When the sole requested video passes the free-edit delivery path, set `paymentStatus: "paid"` (and document in history if needed) on create, or add an explicit `isComplimentary` flag consumed by admin list filters. Keep eligibility checks server-side; do not rely on client price display.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Complimentary edit requests created with default unpaid payment status | HIGH | Open | src/services/edit-request.service.ts | 201-210 |

**Merge readiness:** Request changes — set payment (or equivalent) for in-window complimentary creates before merge.
