# PR review — skillshow-admin-ui #362 (SKSH-433)

**Repo:** SkillshowFx/skillshow-admin-ui  
**Branch:** SKSH-433 → main  
**Head:** `2363310c4aa50776a743e3fb60bd7f2693b083bb`  
**Scope:** Open edit modal from My Videos list (nav state `openEdit`); shared actions dropdown desktop + mobile  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`  
**Updated:** 2026-07-27 — re-verify; findings #1–#2 ✅ Fixed

## GitHub comments

_(none — no Open Critical/High/Medium)_

## Findings

---
Responsive/mobile list path missing Edit

Risk Level: HIGH
File Path: src/pages/videos/my-videos/dashboard/components/my-videos-table.tsx
Lines: 146-156

Description:
**Global consistency.** Desktop had Edit; mobile `MyVideosTableResponsive` / `MobileVideoCard` originally did not.

**Re-verify:** `canEditVideo` / `onEdit` thread through table → responsive → `MobileVideoCard`. Shared `getVideoListActionDropdownItems` used on both surfaces.

Impact:
- (Resolved) Mobile/narrow layout can open edit from the list.

Recommendation:
_(done)_
---

---
openEdit left in history state after consume

Risk Level: HIGH
File Path: src/pages/videos/details/index.tsx
Lines: 164-173

Description:
**Contract / UX.** Prior head opened edit once per mount but left `openEdit` in history.

**Re-verify:** After open, `navigate(..., { replace: true, state: withoutOpenEditLocationState(location.state) })` clears the one-shot flag.

Impact:
- (Resolved) Refresh no longer re-opens edit from stale history state.

Recommendation:
_(done)_
---

**Positive notes:** Shared action menu util + `VIDEO_LIST_ACTION_KEYS`; `canEditVideo` gates Edit; delete still via `DestructiveActionConfirmModal`.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Responsive/mobile list path missing Edit | HIGH | ✅ Fixed | src/pages/videos/my-videos/dashboard/components/my-videos-table.tsx | 146-156 |
| 2 | openEdit left in history state after consume | HIGH | ✅ Fixed | src/pages/videos/details/index.tsx | 164-173 |

**Merge readiness:** No open Critical/High/Medium blockers.
