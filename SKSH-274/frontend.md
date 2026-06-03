# Frontend PR Review — skillshow-admin-ui (`SKSH-274`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-274`  
**Base:** `main...HEAD`  
**Initial review:** 2026-05-29  
**Re-reviewed:** 2026-06-03 — merge at `8f0b36fc` (server-side export-all, events refactor, partners/crew/skillshow-users/admin edit request)  
**Scope:** Export All CSV on list dashboards (Critical / High / Medium only)

**Files changed (vs `main`):** 21 files — events, partners, video library, management users, admin edit requests, shared `fetch-all-paginated-list`, `EXPORT_ALL_LABEL`  
**Findings:** 4 total — 3 from initial review (**all fixed**), **1 new Medium** (dead code)

---

## Overview

Export All is implemented on server-paginated lists via `handleExportAll` + `fetchAllPaginatedListItems` / `fetchAllEventsListItems` (events, partners, crew users, skillshow users, admin edit requests). Video library and app users keep client-side filtered export via `onExportAllReady`. Teams export UI from the first commit was removed; `export-teams-csv.ts` remains unused.

---

## Findings (initial review)

---
Teams “Export All” ignores search unlike other dashboards

Risk Level: MEDIUM  
File Path: src/pages/teams/index.tsx  
Lines: (removed)

Description:
On the first revision, teams **Export All** exported the full `teams` array while **Export Teams** respected search.

Impact:
- Users could export rows outside the filtered card grid.

Recommendation:
Align **Export All** with `filteredTeams`, or remove teams export from this PR.

**Re-review:** ✅ **Fixed** — teams export buttons were removed from `teams/index.tsx`; the page no longer exposes export actions (teams dropped from this PR’s UX scope).

---

---
Teams export buttons redundant when search is empty

Risk Level: MEDIUM  
File Path: src/pages/teams/index.tsx  
Lines: (removed)

Description:
With no search, **Export Teams** and **Export All** exported identical data.

Impact:
- Duplicate controls with no functional difference.

Recommendation:
Show a single export action or hide **Export All** until search is active.

**Re-review:** ✅ **Fixed** — both export buttons removed with teams export scope.

---

---
Events export-all wired through table props though parent already owns `filteredRows`

Risk Level: MEDIUM  
File Path: src/pages/events/dashboard/index.tsx  
File Path: src/pages/events/dashboard/components/events-table.tsx

Description:
`filteredRows` and `onExportAllReady` were passed into `EventsTable` only to register export-all.

Impact:
- Extra props and child `useEffect` for logic the page already owned.

Recommendation:
Colocate `handleExportAll` in `index.tsx` and remove table export-all wiring.

**Re-review:** ✅ **Fixed** — `handleExportAll` lives in `index.tsx` (lines 120–140) and calls `fetchAllEventsListItems` with `debouncedSearch`, `eventTypeFilter`, and `listingStatus`. `EventsTable` only registers page export via `onExportReady`; `onExportAllReady` and `filteredRows` props are gone from `types.ts` / table.

---

## Findings (re-review)

---
Unused `export-teams-csv` module left in tree

Risk Level: MEDIUM  
File Path: src/pages/teams/utils/export-teams-csv.ts  
Lines: 1-25

Description:
`exportTeamsAsCsv` is defined but has no imports anywhere in the repo. Teams list UI no longer wires export after the follow-up changes.

Impact:
- Dead code shipped with the PR; confuses future readers and suggests incomplete teams export work.

Recommendation:
Delete `src/pages/teams/utils/export-teams-csv.ts`, or wire it when teams export is product-ready.

**PR comment:** **Medium:** `export-teams-csv.ts` is unused after teams export was removed—please delete the file or hook it up.

---

## Positive notes

- Server-paginated export-all correctly applies current filters (search, tab, type/role) and fetches all pages via `fetchAllPaginatedListItems` / `fetchAllEventsListItems`.
- Partners uses a single `PartnersTable` instance with page-level `handleExportAll` (no duplicate tab handlers).
- Events export applies `tableSort` to the full fetched set before CSV generation.
- Shared `EXPORT_ALL_LABEL` constant improves label consistency on most pages.
- Export warnings migrated to Sonner `toast` on events table (aligned with video library).

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Teams Export All ignores search | MEDIUM | ✅ Fixed | src/pages/teams/index.tsx | — |
| 2 | Teams export buttons redundant when search empty | MEDIUM | ✅ Fixed | src/pages/teams/index.tsx | — |
| 3 | Events export-all prop drill through table | MEDIUM | ✅ Fixed | src/pages/events/dashboard/index.tsx | 120-140 |
| 4 | Unused `export-teams-csv` module | MEDIUM | Open | src/pages/teams/utils/export-teams-csv.ts | 1-25 |

**Re-review verdict:** Original feedback addressed. **Merge after removing dead `export-teams-csv.ts`** (or restoring teams export with correct semantics).
