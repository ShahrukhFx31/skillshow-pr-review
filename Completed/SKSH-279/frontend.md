# Frontend PR Review — skillshow-admin-ui (`SKSH-279`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-279`  
**Base:** `origin/main...HEAD`  
**Initial review:** `283da192` (*fix: ui bugs*)  
**Re-review:** `279d46e1` (*fix: pr changes*)  
**Scope:** Edit Request list UI — welcome banner, stat cards, toolbar, empty-search actions (Critical / High / Medium only)

**Files changed (substantive):** 8 files under `src/pages/editRequest/` (no `global.css` changes on final branch).

---

## Re-review verification (`279d46e1`)

| # | Finding | Resolution |
|---|---------|------------|
| 1 | Card stat labels | `buildEditRequestListSummaryCards()` shared in `edit-request-list-summary.utils.ts`; `EditRequestCards` maps `summaryCards` with distinct titles (lines 67–79). |
| 2 | Empty-search `withStartActions` | `renderList([], true)` in `index.tsx` (line 281). |
| 3 | Stat styles in `global.css` | Removed; `EDIT_REQUEST_STAT_CARD_CLASS` colocated in `editRequest.constants.ts`; table and cards both use it. |

---

---
Mobile/tablet stat cards all labeled “Total Request”

Risk Level: HIGH  
File Path: src/pages/editRequest/components/EditRequestCards.tsx  
Lines: 67-79 (was 70-77)

Description:
On the card layout (`EditRequestCards`), each status summary tile mapped `effectiveStatusCounts` but every card used the same hardcoded title `"Total Request"`.

Impact:
- Misleading metrics on tablet/mobile — users could not tell which count was pending, rejected, etc.
- Desktop table layout showed correct aggregated titles; card layout was inconsistent.

Recommendation:
Mirror `EditRequestTable`’s `summaryCards` aggregation — **done** via `buildEditRequestListSummaryCards`.

**Re-review:** Fixed in `279d46e1`. Cards and table share `edit-request-list-summary.utils.ts`; four KPI tiles with correct titles.

**PR comment (line 77):** Resolved — card layout now uses the same summary aggregation as desktop.

---

---
Empty-search list passes layout flag as `withStartActions`

Risk Level: MEDIUM  
File Path: src/pages/editRequest/index.tsx  
Lines: 281 (was 277-281)

Description:
`renderList([], useCardLayout)` passed the layout flag as `withStartActions`, disabling create flow on desktop after zero-result search.

Impact:
- “New Edit Request” modals did not start the create flow on desktop empty-search.

Recommendation:
`renderList([], true)` — **done**.

**Re-review:** Fixed in `279d46e1` at line 281.

**PR comment (line 281):** Resolved — empty-search branch always enables start actions.

---

---
Edit-request stat card styles added to `global.css`

Risk Level: MEDIUM  
File Path: src/global.css (was 326, 396-405)

Description:
`.stat-card-surface` was added to global CSS for edit-request-only stat cards.

Impact:
- Feature styling in global stylesheet; harder to maintain per project conventions.

Recommendation:
Colocate in feature constants or component — **done** as `EDIT_REQUEST_STAT_CARD_CLASS` in `editRequest.constants.ts`.

**Re-review:** Fixed in `279d46e1`. No `stat-card-surface` / `brand-stat-card` remains in the repo.

**PR comment:** Resolved — stat card class lives in feature constants.

---

## Positive notes

- **Banner extraction:** `EditRequestListBanner` + `EDIT_REQUEST_LIST_PAGE_COPY` matches the `GradientWelcomeBanner` pattern used on other dashboards.
- **Shared summary util:** `buildEditRequestListSummaryCards` removes duplication between table and card layouts and centralizes summary titles.
- **Toolbar cleanup:** Optional `listTitle`; banner owns the page title.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Mobile/tablet stat cards all labeled “Total Request” | HIGH | ✅ Fixed | src/pages/editRequest/components/EditRequestCards.tsx | 67-79 |
| 2 | Empty-search list passes layout flag as `withStartActions` | MEDIUM | ✅ Fixed | src/pages/editRequest/index.tsx | 281 |
| 3 | Edit-request stat card styles in `global.css` | MEDIUM | ✅ Fixed | src/pages/editRequest/constants/editRequest.constants.ts | 66-68 |

**Merge readiness:** No open Critical/High/Medium blockers. All prior findings verified fixed on `279d46e1`.
