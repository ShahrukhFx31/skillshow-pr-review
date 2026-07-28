# Frontend PR Review — skillshow-admin-ui (`SKSH-435`)

**Repo:** SkillshowFx/skillshow-admin-ui  
**PR:** https://github.com/SkillshowFx/skillshow-admin-ui/pull/364  
**Branch:** `SKSH-435` → main  
**Head:** `48da1e9f390bff8991efc81f8c949fa3b7d63061`  
**Scope:** Allow complimentary SkillShow delivery + additional athlete videos in edit-request create UX (lock free-edit, pricing, selection)  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`  
**Paired backend:** `pr-review/SKSH-435/backend.md` (skillshow #253)

## GitHub comments

### `src/pages/editRequest/index.tsx`

- **L244** — Duplicated free-edit lock merge logic across three handlers

## Findings

---
Duplicated free-edit lock merge logic across three handlers

Risk Level: MEDIUM  
**Status:** Open  
File Path: src/pages/editRequest/index.tsx  
Lines: 244-255, 324-360, 369-390

Description:
**DRY.** `handleStartWithVideos`, `UploadVideosScreen.onSelectVideos`, and `EnhanceVideoScreen.onAddVideos` each reimplement the same “keep locked complimentary video + merge non-free extras” rules (including the branch that re-attaches a locked free video when the modal omits it). `SelectFromMyVideosModal` also duplicates lock checks in `toggleSelection` / `handleCheckboxChange`. Drift risk as selection rules evolve.

Impact:
- Future lock/pricing rule changes can fix one path and miss another (create vs enhance vs upload).
- Harder to unit-test selection invariants in one place.

Recommendation:
Extract a pure helper, e.g. `mergeEditRequestVideoSelection(prev, next): EditRequestVideoItem[]`, and reuse it from all three page handlers. Optionally share lock-id derivation with the modal via the same util (`lockedFreeEditIds` from a selection list).
---

**Positive notes:** Pricing uses `paidVideoCount` when a free-edit is present; complimentary row remove is blocked; selection mode stays `multiple` so extras can be added — aligns with backend eligibility change (aside from paymentStatus — see backend #1).

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Duplicated free-edit lock merge logic across three handlers | MEDIUM | Open | src/pages/editRequest/index.tsx | 244-255, 324-360, 369-390 |

**Merge readiness:** No open Critical/High on frontend; Medium DRY open. Backend #253 still has open High paymentStatus finding — coordinate before merge.
