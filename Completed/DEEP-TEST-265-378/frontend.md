# Frontend PR Review — skillshow-admin-ui (`DEEP-TEST`)

**Repo:** SkillshowFx/skillshow-admin-ui  
**PR:** https://github.com/SkillshowFx/skillshow-admin-ui/pull/378  
**Branch:** `deep/test` → main  
**Head:** `8811f4aa0a5692e5f8b98a0ed60dd5fe740a9de9`  
**Scope:** Free-edit family UI, partner sortOrder/contacts, social-platform tabs, permission modal popup container, Ant message positioning  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`  
**Paired:** `pr-review/Completed/DEEP-TEST-265-378/backend.md` (#265)  
**Updated:** 2026-08-20 — finding #4 Accepted; archived as `Completed/DEEP-TEST-265-378`

## GitHub comments

_(none — no Open findings)_

## Findings

---
Protected module changed (`antd.adapter.tsx`)

Risk Level: CRITICAL
File Path: src/theme/adapter/antd.adapter.tsx
Lines: 135-146

Description:
**Protected module.** `antd.adapter.tsx` is no longer modified in this PR (reverted to `<App>{children}</App>`).

Impact:
- Resolved — frozen adapter left alone.

Recommendation:
N/A — fixed on head `8811f4aa`.
---

---
Never-uploaded shells from prior rounds kept on current free-edit round

Risk Level: HIGH
File Path: src/pages/editRequest/utils/edit-request-output-round.utils.ts
Lines: 42-62, 97-106

Description:
**Global consistency.** FE now mirrors backend `emptyOutputBelongsToCurrentFreeEditRound` and admin filters pass `detail.history` into round grouping / capacity counts.

Impact:
- Resolved — prior-round empty shells no longer block Add Output on the active round.

Recommendation:
N/A — fixed on head `8811f4aa`.
---

---
Feature TreeSelect CSS added to `global.css`

Risk Level: MEDIUM
File Path: src/global.css
Lines: 77-81

Description:
Permission TreeSelect scroll rules moved to colocated `permission-modal.css`.

Impact:
- Resolved — feature CSS is no longer in `global.css`.

Recommendation:
N/A — fixed on head `8811f4aa`.
---

---
App-wide Ant message positioning added to `global.css`

Risk Level: MEDIUM
File Path: src/global.css
Lines: 731-736

Description:
**File structure / styling.** After reverting the protected `antd.adapter` toast config, the PR adds `.ant-message { left/right/transform !important }` in `global.css` to pin Ant messages top-right next to Sonner. That is still an app-wide Ant Design override in the global stylesheet (same concern as editing the adapter, different file).

Impact:
- Global CSS owns Ant message layout with `!important`, which is hard to discover and can fight future theme work.
- Workarounds for frozen `antd.adapter` belong in a dedicated theme ticket, not ad-hoc global rules.

Recommendation:
**Accepted** — keep as-is for this PR; revisit in a theme / design-system ticket if needed.
---

**Positive notes:** Empty-shell round ownership is shared with backend. Permission modal popup/scroll fix is colocated. Social platforms tab grouping remains clean.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Protected module changed (`antd.adapter.tsx`) | CRITICAL | ✅ Fixed | src/theme/adapter/antd.adapter.tsx | 135-146 |
| 2 | Never-uploaded shells from prior rounds kept on current free-edit round | HIGH | ✅ Fixed | src/pages/editRequest/utils/edit-request-output-round.utils.ts | 42-62, 97-106 |
| 3 | Feature TreeSelect CSS added to `global.css` | MEDIUM | ✅ Fixed | src/global.css | 77-81 |
| 4 | App-wide Ant message positioning added to `global.css` | MEDIUM | Accepted | src/global.css | 731-736 |

**Merge readiness:** **Merge-ready** — all findings Fixed or Accepted. Paired with backend #265.
