# Frontend PR Review — skillshow-admin-ui (`SKSH-313`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-313`  
**Base:** `main...HEAD` @ `de4d86a2`  
**Re-verified:** 2026-06-08  
**Scope:** Event view crew management (list/add/remove), server sort/pagination for athletes/crew/videos sub-tables (Critical, High, Medium only)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [backend.md](./backend.md)

**Findings:** 4 (1 High, 3 Medium) — **1 Open**, **1 Accepted**, **2 Fixed**

### Protected modules

No changes to `use-server-table-controls.ts`, `pagination-bar.tsx`, `use-pagination.ts`, `table-sort.ts`, `antd.adapter.tsx`, or `destructive-action-confirm-modal.tsx`.

---

## GitHub comments (Open findings)

### 1. `src/pages/events/onboarding/components/event-view/event-view-related-table.tsx` line 35

**Medium (Global consistency):** Server sort works via `applyServerSort`, but column defs still lack controlled `sortOrder` — pass `sortBy`/`sortOrder` from the section into column builders (video-library `columnSortOrder` pattern) so headers reflect active sort after tab switches.

---

---
Unrelated dashboard card subtitles removed (scope / UX regression)

Risk Level: HIGH  
File Path: src/pages/management/app-users/dashboard/index.tsx  
Lines: 101-109 (removed)  
File Path: src/pages/management/crew-users/dashboard/index.tsx  
Lines: 101-109 (removed)  
File Path: src/pages/management/skillshow-users/dashboard/index.tsx  
Lines: 101-109 (removed)  
File Path: src/pages/partners/dashboard/index.tsx  
Lines: 70-78 (removed)  
File Path: src/pages/events/dashboard/index.tsx  
Lines: 144-152 (removed)

Description:
Five dashboard pages drop their `Card` `title` blocks (page name + descriptive subtitle) while this ticket is event-view crew/pagination work. Commit `723c2f03` (“remove unused Typography”) is unrelated to SKSH-313 and removes user-facing helper copy.

Impact:
- Unrelated UX regression on Partners, App Users, Crew Users, Skillshow Users, and Events dashboards bundled into an events PR.
- Reviewers and QA must validate five extra surfaces; rollback risk if the event PR is held.

Recommendation:
Revert the dashboard `title` removals in this branch (restore `Typography` subtitles) or move them to a separate cleanup PR. Keep SKSH-313 limited to `src/pages/events/**` event-view changes and directly related API/utils.

**PR comment (`app-users/dashboard/index.tsx` — re-add at former title site ~line 101):**  
**High:** This PR removes dashboard card title/subtitle copy unrelated to event crew work — please revert on the five management/partner/events dashboard pages or split to a separate PR.

**Status:** Accepted — intentional dashboard title cleanup bundled in this PR per team decision; page context remains clear from route/nav.

---

---
Event view tables omit controlled column `sortOrder` (Global consistency)

Risk Level: MEDIUM  
File Path: src/pages/events/onboarding/components/event-view/event-view-related-table.tsx  
Lines: 35-91  
File Path: src/pages/events/onboarding/components/event-view/event-view-related-columns.tsx  
Lines: 32-86  
File Path: src/pages/events/onboarding/constants.ts  
Lines: 138-190

Description:
**Global consistency:** Server sort state lives in `useServerTableControls` and flows to API via `use-event-view-related-lists`, but Ant Table column defs only set `sorter: true` without per-column `sortOrder`. Elsewhere (`video-library-columns.tsx`) a `columnSortOrder` helper binds header arrows to server `sortBy`/`sortOrder`.

**Re-verification (@ `de4d86a2`):** Commit `de4d86a2` removes unused `sortBy`/`sortOrder` props from `EventViewRelatedTable` (they were never consumed) — that cleans types but does **not** add controlled column `sortOrder`. `buildEventViewAthleteColumns` / `buildEventViewCrewColumns` still take no sort args; `EVENT_VIEW_*_COLUMNS` have no `sortOrder` field. `showSorterTooltip` enables hover tooltips only.

Impact:
- Sorting works server-side but column headers do not reflect active sort after tab switch or initial load; users cannot see which column/direction is applied without inferring from data order.

Recommendation:
Pass `sortBy` / `sortOrder` from `event-view-related-section.tsx` into column builders and set `sortOrder: columnSortOrder(key, sortBy, sortOrder)` on each sortable column — same pattern as `video-library-columns.tsx`.

**PR comment (`event-view-related-table.tsx` line 35):**  
**Medium (Global consistency):** Server sort is wired, but columns still lack controlled `sortOrder` — pass sort state into column builders (`columnSortOrder` pattern) so headers show the active sort after tab changes. Removing unused table props (`de4d86a2`) does not fix header indicators.

**Status:** Open

---

---
Add Crew modal duplicates Add Athlete modal (DRY)

Risk Level: MEDIUM  
File Path: src/pages/events/onboarding/components/event-view/event-view-add-related-modal.tsx  
Lines: 1-78  
File Path: src/pages/events/onboarding/hooks/use-event-view-add-related-modal.ts  
Lines: 1-62

Description:
**DRY.** `c3c73a14` extracts `EventViewAddRelatedModal` + `useEventViewAddRelatedModal`; athlete and crew modals are thin wrappers passing service fns, query keys, and copy constants.

**Re-verification (@ `de4d86a2`):** Unchanged; shared modal/hook still in place.

**Status:** ✅ Fixed

---

---
Backend crew filters exist; event view filter UI still placeholder

Risk Level: MEDIUM  
File Path: src/pages/events/onboarding/components/event-view/event-view-related-actions.tsx  
Lines: 1-22

Description:
Backend supports crew `designation`/`status` filters, but the event view had a non-functional filter dropdown. **Re-verification (@ `de4d86a2`):** Filter control removed from `EventViewRelatedActions` (search-only toolbar); no misleading filter affordance.

**Status:** ✅ Fixed

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Unrelated dashboard card subtitles removed | HIGH | Accepted | src/pages/management/app-users/dashboard/index.tsx | 101-109 |
| 2 | Event view tables omit controlled column `sortOrder` | MEDIUM | Open | src/pages/events/onboarding/components/event-view/event-view-related-table.tsx | 35-91 |
| 3 | Add Crew modal duplicates Add Athlete modal | MEDIUM | ✅ Fixed | src/pages/events/onboarding/components/event-view/event-view-add-related-modal.tsx | 1-78 |
| 4 | Backend crew filters exist; filter UI placeholder | MEDIUM | ✅ Fixed | src/pages/events/onboarding/components/event-view/event-view-related-actions.tsx | 1-22 |

### Re-review notes (2026-06-08 @ `de4d86a2`)

| Change | Verdict |
|--------|---------|
| `de4d86a2` remove unused `sortBy`/`sortOrder` table props | Cleanup only — **does not fix** column header indicators |
| `c3c73a14` shared add-related modal/hook | **Fixed** DRY finding |
| Filter dropdown removed from related actions | **Fixed** placeholder filter finding |
| Dashboard `Typography` titles removed | **Accepted** per team |
| Backend `b63c391` `crewNameSort` | See [backend.md](./backend.md) — all backend findings fixed |

**Positive notes:** Event related section uses `useServerTableControls`, `applyServerSort`, `PaginationBar`, and `DestructiveActionConfirmModal`; tab switch resets search and per-tab default sort; crew CRUD API-driven; mock crew rows removed; shared add-related modal reduces duplication.

**Merge readiness:** **Not merge-ready** — 1 open Medium on frontend (controlled column `sortOrder`). Backend has no open blockers.
