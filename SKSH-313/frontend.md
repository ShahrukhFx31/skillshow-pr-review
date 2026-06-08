# Frontend PR Review — skillshow-admin-ui (`SKSH-313`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-313`  
**Base:** `main...HEAD` (18 files, ~663 insertions / ~381 deletions)  
**Scope:** Event view crew management (list/add/remove), server sort/pagination for athletes/crew/videos sub-tables, shared table controls refactor (Critical, High, Medium)  
**Findings:** 4 (1 High, 3 Medium)

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

Description:
Four management dashboard pages drop their `Card` `title` blocks (page name + descriptive subtitle) while this ticket is event-view crew/pagination work. Commit `723c2f03` (“remove unused Typography”) is unrelated to SKSH-313 and removes user-facing helper copy (e.g. “Manage athletes, parents, and coaches…”, “Search by name or email…”).

Impact:
- Unrelated UX regression on Partners, App Users, Crew Users, and Skillshow Users dashboards bundled into an events PR.
- Reviewers and QA must validate four extra surfaces; rollback risk if the event PR is held.

Recommendation:
Revert the dashboard `title` removals in this branch (restore `Typography` subtitles) or move them to a separate cleanup PR. Keep SKSH-313 limited to `src/pages/events/**` and directly related API/utils changes.

**PR comment (`app-users/dashboard/index.tsx` — re-add at former title site ~line 101):**  
**High:** This PR removes the App Users card title/subtitle unrelated to event crew work — please revert these dashboard title changes on the four management/partner pages or split to a separate PR.

---

---
Event view tables omit controlled column `sortOrder` (Global consistency)

Risk Level: MEDIUM  
File Path: src/pages/events/onboarding/components/event-view/event-view-related-table.tsx  
Lines: 35-91  
File Path: src/pages/events/onboarding/constants.ts  
Lines: 96-145

Description:
**Global consistency:** Server sort state (`sortBy`, `sortOrder`) is passed into `EventViewRelatedTable`, but column defs only set `sorter: true` without per-column `sortOrder`. Elsewhere (e.g. `video-library-columns.tsx`) columns use a `columnSortOrder` helper so Ant Table shows the active sort direction and matches server state after tab switches.

Impact:
- Sorting works server-side but column headers do not reflect active sort; users cannot see which column/direction is applied without inferring from data order.

Recommendation:
Pass `sortBy` / `sortOrder` into column builders (or map `EVENT_VIEW_*_COLUMNS` with a shared `columnSortOrder` helper from `@/utils` or colocated util) so each sortable column sets `sortOrder` when it matches the active server sort — same pattern as video library.

**PR comment (`event-view-related-table.tsx` line 53):**  
**Medium (Global consistency):** Tables wire `applyServerSort` but columns lack controlled `sortOrder` — mirror `video-library-columns.tsx` so headers show the active server sort after tab/sort changes.

---

---
Add Crew modal duplicates Add Athlete modal (DRY)

Risk Level: MEDIUM  
File Path: src/pages/events/onboarding/components/event-view/event-view-add-crew-modal.tsx  
Lines: 1-102  
File Path: src/pages/events/onboarding/components/event-view/event-view-add-athlete-modal.tsx  
Lines: 1-107

Description:
**DRY:** `EventViewAddCrewModal` and `EventViewAddAthleteModal` are nearly identical: debounced search state, `useQuery` for available list, `useMutation` for add, Ant `Modal` + multi-select `Form.Item`, same `EVENTS_LIST_MAX_LIMIT` / toast for zero adds. Only labels, query keys, and service calls differ.

Impact:
- Search debounce, empty-query gating, and add-flow bugfixes must be applied twice; easy drift between athlete and crew modals.

Recommendation:
Extract a small shared `EventViewAddRelatedModal` (props: `title`, `label`, `listAvailable`, `addItems`, `mapOption`, `queryKeyFactory`) or a `useEventViewAddRelatedModal` hook; keep thin athlete/crew wrappers for copy and API wiring.

**PR comment (`event-view-add-crew-modal.tsx` line 10):**  
**Medium (DRY):** Add Crew modal mirrors Add Athlete almost line-for-line — consider a shared add-related modal/hook so search, debounce, and add flows stay in one place.

---

---
Backend crew filters exist; event view filter UI still placeholder

Risk Level: MEDIUM  
File Path: src/pages/events/onboarding/hooks/use-event-view-related-lists.ts  
Lines: 35-44  
File Path: src/pages/events/onboarding/components/event-view/event-view-related-actions.tsx  
Lines: 37 (unchanged — not commentable on GitHub)

Description:
Backend `eventCrewListQuerySchema` supports `designation` and `status` filters, but the event view filter dropdown still renders “No filters for this preview.” (`event-view-related-actions.tsx` line 37 — **pre-existing, unchanged in this PR**) and `listParams` never sends `designation` or `status`. Crew (and athlete/video) sub-tables expose a filter affordance with no wiring.

Impact:
- Admins cannot filter crew by designation/status despite API support; filter button sets false expectations.

Recommendation:
Either wire designation/status into `useServerTableControls` `extraFilterState` + `listParams` for the Crew tab (and athlete/video if applicable), or hide/disable the filter control until filters are implemented.

**PR comment — use a changed line in the diff (GitHub cannot comment on unchanged line 37):**

| File | Line (branch) | Why it's in the PR diff |
|------|---------------|-------------------------|
| `src/pages/events/onboarding/hooks/use-event-view-related-lists.ts` | **58** | New `listEventCrew` call uses `listParams` with no `designation`/`status` |
| `src/pages/events/onboarding/hooks/use-event-view-related-lists.ts` | **35-42** | New shared `listParams` object — only search/sort/pagination |
| `src/pages/events/onboarding/components/event-view/event-view-related-section.tsx` | **217** | Renders `EventViewRelatedActions` with `filterOpen` while crew API filters are added |
| `src/pages/events/onboarding/components/event-view/event-view-related-actions.tsx` | **16** | Only line changed in this file (`allowClear`) — weak anchor; prefer `hooks.ts` |

**Suggested inline comment (`use-event-view-related-lists.ts` line 58):**  
**Medium:** Crew list API now supports `designation`/`status`, but `listParams` only sends search/sort — wire filters into `extraFilterState` + query params for Crew, or hide the filter button (`event-view-related-actions.tsx` placeholder is unchanged and not commentable in this diff).

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Unrelated dashboard card subtitles removed | HIGH | Open | src/pages/management/app-users/dashboard/index.tsx | 101-109 |
| 2 | Event view tables omit controlled column `sortOrder` | MEDIUM | Open | src/pages/events/onboarding/components/event-view/event-view-related-table.tsx | 35-91 |
| 3 | Add Crew modal duplicates Add Athlete modal | MEDIUM | Open | src/pages/events/onboarding/components/event-view/event-view-add-crew-modal.tsx | 1-102 |
| 4 | Backend crew filters exist; filter UI placeholder | MEDIUM | Open | src/pages/events/onboarding/hooks/use-event-view-related-lists.ts | 35-44, 58 |

**Positive notes:** Event related section correctly uses `useServerTableControls` with `activeTab` in `filterKey`, `applyServerSort`, `PaginationBar`, and `DestructiveActionConfirmModal` for athlete/crew remove; tab switch resets search and per-tab default sort; `queryKey` includes sort params; crew CRUD wired through `eventService` with `crewUserId` on rows; mock `EVENT_VIEW_CREW_ROWS` removed in favor of API-driven crew list.

**Merge readiness:** **Not merge-ready** — 1 open High and 3 open Medium findings (plus backend blockers in `backend.md`).
