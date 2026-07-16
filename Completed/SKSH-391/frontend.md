# PR review (SKSH-391) — skillshow-admin-ui

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow-admin-ui` |
| PR | [#341](https://github.com/SkillshowFx/skillshow-admin-ui/pull/341) |
| Branch | `SKSH-391` → `main` |
| Head | `e220edb3206f8c08e252eb5992309f1b8a8a09a2` |
| Scope | Visibility column on My Videos + shared `VideoVisibilityCue` for mobile |
| Prompts | `pr-review/prompts/frontend-system-prompt.md` |
| Re-verify | 2026-07-15 — head `e220edb3` (mobile cue + shared component) |

### Protected modules

| Module | Status |
|--------|--------|
| pagination / table-sort / audit-log / antd.adapter | **Not modified** ✅ |

### Positive notes

- Reuses `isVideoPublic` / `resolveVideoVisibilityLabel` via shared `VideoVisibilityCue`.
- No bogus `sorter` on UI-only `visibility` key — avoids broken `sortBy` against server allow-list.
- Desktop column and `MobileVideoCard` both consume the same cue component.

## GitHub comments

_No open findings._

## Findings

---
Visibility missing on mobile My Videos cards

Risk Level: MEDIUM
File Path: src/pages/videos/my-videos/dashboard/components/my-videos-columns.tsx
Lines: 86

Description:
**Global consistency** — Visibility was desktop-only; mobile card lacked Public/Private labeling.

**Re-verify (e220edb3):** ✅ Fixed — extracted `VideoVisibilityCue`; desktop column and `MobileVideoCard` both render it.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Visibility missing on mobile My Videos cards | MEDIUM | ✅ Fixed | `src/pages/videos/my-videos/dashboard/components/my-videos-columns.tsx` | 86 |

**Merge readiness:** No open Critical/High/Medium blockers.
