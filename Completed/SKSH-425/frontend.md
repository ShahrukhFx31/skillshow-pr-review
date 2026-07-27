# PR review — skillshow-admin-ui #359 (SKSH-425)

**Repo:** SkillshowFx/skillshow-admin-ui  
**Branch:** SKSH-425 → main  
**Head:** `24dfa213099deed1c3ad5ada4df9ed33f7722b14`  
**Scope:** Collapsible `GradientWelcomeBanner` (controlled/uncontrolled open, AnimatePresence)  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`  
**Updated:** 2026-07-27 — re-verify; findings #1–#2 fixed (`collapsible = false`; `rightSlot` always visible)

## GitHub comments

_(none — no Open Critical/High/Medium)_

## Findings

---
collapsible defaults to true and hides rightSlot/bottomSlot globally

Risk Level: HIGH
File Path: src/components/welcome/gradient-welcome-banner.tsx
Lines: 36

Description:
**Global consistency / UX.** Prior head defaulted `collapsible = true` and hid `rightSlot` when collapsed.

**Re-verify:** Default is now `collapsible = false` (opt-in). `contentOpen = !collapsible || open`. `rightSlot` always renders when provided; only description/`bottomSlot` collapse.

Impact:
- (Resolved) Existing banner call sites keep prior behavior until they opt in.

Recommendation:
_(done)_
---

---
Collapse should not unmount primary rightSlot actions

Risk Level: HIGH
File Path: src/components/welcome/gradient-welcome-banner.tsx
Lines: 105-119

Description:
**UX / KISS.** Prior head gated `rightSlot` on `open`.

**Re-verify:** `rightSlot` is always rendered beside the chevron; collapse only affects description/`bottomSlot`.

Impact:
- (Resolved) CTAs/nav in `rightSlot` remain reachable while collapsed.

Recommendation:
_(done)_
---

**Positive notes:** Opt-in collapse; `aria-expanded` / `aria-label`; `cn()` for classes.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | collapsible defaults to true and hides rightSlot/bottomSlot globally | HIGH | ✅ Fixed | src/components/welcome/gradient-welcome-banner.tsx | 36 |
| 2 | Collapse should not unmount primary rightSlot actions | HIGH | ✅ Fixed | src/components/welcome/gradient-welcome-banner.tsx | 105-119 |

**Merge readiness:** No open Critical/High/Medium blockers.
