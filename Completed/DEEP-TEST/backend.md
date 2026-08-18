# Backend PR Review — skillshow (`DEEP-TEST`)

**Repo:** SkillshowFx/skillshow  
**PR:** https://github.com/SkillshowFx/skillshow/pull/264  
**Branch:** `deep/test` → main  
**Head:** `953c7a7c79f28cebd406cc3b357a892525a7fb51`  
**Scope:** Complimentary free-edit reopen rounds, partner sort order / SportsRecruits delete guard, crew admin profile fields, video family lineage  
**Prompt:** `pr-review/prompts/backend-system-prompt.md`  
**Paired:** `pr-review/Completed/DEEP-TEST/frontend.md` (#377)  
**Updated:** 2026-08-18 — re-verify on head `953c7a7c` (findings #1–#2 fixed)

## GitHub comments

_(none — no Open Critical/High findings)_

## Findings

---
Mixed free-edit reopen keeps paid status when extras are added

Risk Level: HIGH
File Path: src/services/edit-request.service.ts
Lines: 413

Description:
**KISS / billing.** `reopen` now sets `paymentStatus` to `"unpaid"` whenever additional videos are included (`additionalIds.length === 0 ? "paid" : "unpaid"`). Covered by `edit-request.service.test.ts`.

Impact:
- Resolved — mixed complimentary reopen with extra athlete uploads is unpaid.

Recommendation:
N/A — fixed on head `953c7a7c`.
---

---
Admin output list drops outputs with no versions

Risk Level: HIGH
File Path: src/services/edit-request-output.service.ts
Lines: 440

Description:
**Global consistency.** `mapAdminOutputsWithBatchLoad` returns all mapped outputs (empty `versions` included). `buildOutputsForAdminDetail keeps outputs with no versions` test covers this.

Impact:
- Resolved — empty admin output slots remain after refetch.

Recommendation:
N/A — fixed on head `953c7a7c`.
---

**Positive notes:** Reopen is owner-scoped via `findByIdAndUserId`, source videos are ownership-checked, and the 7-day window is enforced on the root library delivery. Partner audit still goes through `auditLogService`. SportsRecruits delete guard is centralized.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Mixed free-edit reopen keeps paid status when extras are added | HIGH | ✅ Fixed | src/services/edit-request.service.ts | 413 |
| 2 | Admin output list drops outputs with no versions | HIGH | ✅ Fixed | src/services/edit-request-output.service.ts | 440 |

**Merge readiness:** **Merge-ready** — ship with frontend #377.
