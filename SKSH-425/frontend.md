# PR review — skillshow-admin-ui #359 (SKSH-425)

**Repo:** SkillshowFx/skillshow-admin-ui  
**Branch:** SKSH-425 → main  
**Head:** `6e803107612a70f0d86172e49c1ba6045654f486`  
**Scope:** Collapsible `GradientWelcomeBanner` (controlled/uncontrolled open, AnimatePresence)  
**Prompt:** `pr-review/prompts/frontend-system-prompt.md`

## GitHub comments

### `src/components/welcome/gradient-welcome-banner.tsx`

- **L36** — `collapsible` defaults to true and hides `rightSlot` / `bottomSlot` for all consumers
- **L105** — Collapse should not unmount primary `rightSlot` actions

## Findings

---
collapsible defaults to true and hides rightSlot/bottomSlot globally

Risk Level: HIGH
File Path: src/components/welcome/gradient-welcome-banner.tsx
Lines: 36, 105-119, 144-155

Description:
**Global consistency / UX.** `GradientWelcomeBanner` is used across many pages (teams, videos, dashboard, management lists, etc.). This PR defaults `collapsible = true` and, when collapsed, unmounts `description`, `rightSlot`, and `bottomSlot`. Call sites that put navigation/CTAs in `rightSlot` (e.g. team details “Back to Teams”, import-tool help, dashboard profile-completion) lose those actions until the user expands again. No callers in this PR set `collapsible={false}` to opt out.

Impact:
- Shared banner behavior changes for every existing page without per-call-site review.
- Primary/navigation actions in `rightSlot` disappear while collapsed.

Recommendation:
Prefer backward-compatible default:

```ts
collapsible = false,
```

Opt in where product wants collapse. If default-on is required, keep `rightSlot` visible when collapsed and only hide description/`bottomSlot`.
---

---
Collapse should not unmount primary rightSlot actions

Risk Level: HIGH
File Path: src/components/welcome/gradient-welcome-banner.tsx
Lines: 105-119

Description:
**UX / KISS.** Even with an opt-in collapse, gating `rightSlot` on `open` couples “less descriptive chrome” with “hide actions.” Collapse is for density; actions should remain reachable. Title already stays visible; the chevron can sit beside a still-visible `rightSlot`.

Impact:
- Collapsed state removes CTAs/nav from the banner row.

Recommendation:
Always render `rightSlot` when provided; only animate/hide description and `bottomSlot` with `open`.
---

**Positive notes:** Controlled vs uncontrolled open pattern is clear; `aria-expanded` / `aria-label` on the toggle; `cn()` used for class merging.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | collapsible defaults to true and hides rightSlot/bottomSlot globally | HIGH | Open | src/components/welcome/gradient-welcome-banner.tsx | 36, 105-119, 144-155 |
| 2 | Collapse should not unmount primary rightSlot actions | HIGH | Open | src/components/welcome/gradient-welcome-banner.tsx | 105-119 |

**Merge readiness:** Request changes — open High findings #1–#2.
