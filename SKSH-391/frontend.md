# PR review (SKSH-391) — skillshow-admin-ui

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow-admin-ui` |
| PR | [#341](https://github.com/SkillshowFx/skillshow-admin-ui/pull/341) |
| Branch | `SKSH-391` → `main` |
| Head | `fae28dd2c40fd4c8b7bf7413997cd0d91169d31d` |
| Scope | Desktop Visibility column on My Videos dashboard |
| Prompts | `pr-review/prompts/frontend-system-prompt.md` |
| Re-verify | 2026-07-14 — head unchanged `fae28dd2`; Medium finding still Open |

### Protected modules

| Module | Status |
|--------|--------|
| pagination / table-sort / audit-log / antd.adapter | **Not modified** ✅ |

### Positive notes

- Reuses `isVideoPublic` / `resolveVideoVisibilityLabel`.
- No bogus `sorter` on UI-only `visibility` key — avoids broken `sortBy` against server allow-list.
- Column key added to `MY_VIDEOS_COLUMN_KEYS`.

## GitHub comments

### `src/pages/videos/my-videos/dashboard/components/my-videos-columns.tsx`

- **L86** — Visibility missing on mobile My Videos cards (MEDIUM)

## Findings

---
Visibility missing on mobile My Videos cards

Risk Level: MEDIUM
File Path: src/pages/videos/my-videos/dashboard/components/my-videos-columns.tsx
Lines: 86

Description:
**Global consistency** — Visibility is added only on the desktop column path. The same hook still renders `MobileVideoCard` for `!isMdUp`, and that card was not updated.

Impact:
- Phone/tablet My Videos users cannot see Public/Private labeling introduced on desktop.
- Desktop vs mobile labeling can drift.

Recommendation:
Surface the same visibility affordance in `MobileVideoCard` (icon + `resolveVideoVisibilityLabel(row.isPublic)`), ideally via a small shared cell reused by desktop and mobile.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Visibility missing on mobile My Videos cards | MEDIUM | Open | `src/pages/videos/my-videos/dashboard/components/my-videos-columns.tsx` | 86 |

**Merge readiness:** No Critical/High blockers; 1 Medium open (mobile parity).
