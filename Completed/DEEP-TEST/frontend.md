# Frontend PR Review — skillshow-admin-ui (`DEEP-TEST`)

**Repo:** SkillshowFx/skillshow-admin-ui  
**PR:** https://github.com/SkillshowFx/skillshow-admin-ui/pull/377  
**Branch:** `deep/test` → main  
**Head:** `19475e35396463697ce677fc590fc41042e56b29`  
**Scope:** Free-edit family list + reopen modal, admin/athlete output round grouping, partner sort order / API connect, crew profile attributes, public policy paths  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`  
**Paired:** `pr-review/Completed/DEEP-TEST/backend.md` (#264)  
**Updated:** 2026-08-18 — re-verify on head `19475e35` (findings #1–#3 fixed)

## GitHub comments

_(none — no Open Critical/High/Medium findings)_

## Findings

---
Current-round empty outputs are dropped from admin round filter

Risk Level: HIGH
File Path: src/pages/adminEditRequest/utils/output/admin-edit-request-output.utils.ts
Lines: 77-86

Description:
**Global consistency.** `filterAdminOutputsToFreeEditRound` now calls shared `filterOutputsToFreeEditRound` with `includeEmptyOnCurrentRound: true`. Empty Add-output rows stay on the active round.

Impact:
- Resolved — current-round empty slots remain visible for add/delete.

Recommendation:
N/A — fixed on head `19475e35`.
---

---
Duplicate free-edit round grouping on admin vs athlete

Risk Level: MEDIUM
File Path: src/pages/editRequest/utils/edit-request-output-round.utils.ts
Lines: 40-83

Description:
**DRY.** Admin `groupAdminOutputsByFreeEditRound` / `filterAdminOutputsToFreeEditRound` now wrap the shared athlete helpers with `includeEmptyOnCurrentRound: true`.

Impact:
- Resolved — one grouping implementation; empty-slot behavior is an explicit flag.

Recommendation:
N/A — fixed on head `19475e35`.
---

---
Client reopen fallback ignores the 7-day trial window

Risk Level: MEDIUM
File Path: src/pages/editRequest/utils/edit-request-reopen-eligibility.utils.ts
Lines: 38-44

Description:
When `reopenEligibility` is missing, the client now returns `canReopen: false` with `reason: "expired"` instead of defaulting the CTA on.

Impact:
- Resolved — expired/unknown window does not show Reopen until the API eligibility payload says so.

Recommendation:
N/A — fixed on head `19475e35`.
---

**Positive notes:** Policy routes were centralized (`policy-paths.ts` / `PUBLIC_POLICY_PATHS`). Partner API-connect is routed through `partner-api-connect.utils`. Reopen modal reuses existing select/upload video modals.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Current-round empty outputs are dropped from admin round filter | HIGH | ✅ Fixed | src/pages/adminEditRequest/utils/output/admin-edit-request-output.utils.ts | 77-86 |
| 2 | Duplicate free-edit round grouping on admin vs athlete | MEDIUM | ✅ Fixed | src/pages/editRequest/utils/edit-request-output-round.utils.ts | 40-83 |
| 3 | Client reopen fallback ignores the 7-day trial window | MEDIUM | ✅ Fixed | src/pages/editRequest/utils/edit-request-reopen-eligibility.utils.ts | 38-44 |

**Merge readiness:** **Merge-ready** — ship with backend #264.
