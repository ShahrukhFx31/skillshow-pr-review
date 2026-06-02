# SKSH-277 — Frontend review (`skillshow-admin-ui`)

**Branch:** `sksh-277`  
**Base:** `main`  
**Re-review:** 2026-06-01 (after `feb3197f` — *fix: feedbacks*)

## Overview

Frontend work covers SKSH-277 (connections filter popover, refetch overlay, pagination guard) and SKSH-280 (account dropdown role gating, i18n). Commit `feb3197f` addresses prior review feedback by removing unreachable policies UI from the account dropdown.

---

## Findings

---
Dead policies modal after menu item removal

Risk Level: MEDIUM  
File Path: src/layouts/components/account-dropdown.tsx  
Lines: (removed)

Description:
Initial review flagged `policiesOpen`, policy imports, and `ResponsiveModal` left behind after the Policies menu item was removed.

Impact:
- Unreachable code and lost header policy shortcut (if still required).

Recommendation:
N/A — resolved in `feb3197f`.

**Re-review:** Removed `ResponsiveModal`, `POLICY_TYPES` / `POLICY_TYPE_LABELS` imports, `policiesOpen` state, and the Policies menu path. Dropdown now only exposes Account Settings and Logout plus optional Complete Profile CTA.

---

---
Complete Profile limited to athlete/crew in header dropdown

Risk Level: MEDIUM  
File Path: src/layouts/components/account-dropdown.tsx  
Lines: 40-44, 59-60

Description:
`showCompletion` requires `isAthleteOrCrew && !isAdmin`. Parent/coach users no longer see **Complete Profile** in the header dropdown when `profileCompletion.percentage < 100`.

Impact:
- Header shortcut hidden for parent/coach; dashboard `WelcomeBanner` still shows `ProfileCompletionCircle` for non-admin roles on dashboards that pass `profileCompletionPercentage` (unchanged vs `main`).

Recommendation:
Intentional for SKSH-280. Optional: one-line comment above `showCompletion` documenting that header CTA is athlete/crew-only.

**Re-review:** No code change; behavior matches SKSH-280 intent. Policies access remains via `/policies/:policyType` and registration links.

---

## Positive notes

- **Feedback addressed:** `fix: feedbacks` cleans up account dropdown dead code.
- **`UserRole` enum** uses `UserRole.Admin` / `UserRole.Crew` instead of string literals.
- **Connections panel:** refetch overlay + `disabled={isFetching}` on pagination; filter popover with page reset via existing hooks.
- **Dashboard component diffs** are mostly Prettier/formatting; no behavioral regressions spotted in `WelcomeBanner`, `RecentVideos`, or `ProfileSnapshot`.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Dead policies modal after menu item removal | MEDIUM | ✅ Fixed | src/layouts/components/account-dropdown.tsx | — |
| 2 | Complete Profile limited to athlete/crew in header | MEDIUM | Accepted | src/layouts/components/account-dropdown.tsx | 40-44, 59-60 |

**Merge readiness:** No open Critical/High/Medium blockers. Optional follow-up: comment on `showCompletion` intent if the team wants it documented in code.
