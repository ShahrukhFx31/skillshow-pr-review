# Frontend PR Review — skillshow-admin-ui (`SKSH-274`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-274`  
**Base:** `main...HEAD`  
**Scope:** Export All CSV on events, partners, video library, and teams (Critical / High / Medium only)

**Files changed:** 13 files across events, partners, video library, and teams dashboards  
**Findings:** 3 (0 Critical, 0 High, 3 Medium)

---

---
Teams “Export All” ignores search unlike other dashboards

Risk Level: MEDIUM
File Path: src/pages/teams/index.tsx
Lines: 46-60, 64-68

Description:
On events, partners, and video library, **Export All** exports the full **filtered** result set (active/inactive tab, search, type/format filters) across all pages. On teams, **Export Teams** uses `filteredTeams` (respects search) while **Export All** calls `exportTeamsAsCsv(teams)` and always exports the full API list, ignoring the search box.

Impact:
- A user filtering by team name and clicking **Export All** gets more rows than the list they are viewing, with no indication search was bypassed
- Cross-page behavior is inconsistent and easy to misread as “export everything on screen / in this filter”

Recommendation:
Align with other pages: make **Export All** export `filteredTeams` (all matches for the current search), and reserve a separate label (e.g. **Export entire list**) only if product truly needs search-ignored export. Example:

```tsx
const exportAllTeams = useCallback(() => {
  if (filteredTeams.length === 0) {
    message.warning("No teams to export");
    return;
  }
  exportTeamsAsCsv(filteredTeams);
}, [filteredTeams]);
```

**PR comment (line 54):** **Medium:** Export All on teams exports the full `teams` array and ignores search; other dashboards export the filtered set. Please align semantics (or rename) so users don’t get unexpected rows.

---

---
Teams export buttons are redundant when search is empty

Risk Level: MEDIUM
File Path: src/pages/teams/index.tsx
Lines: 46-68

Description:
When `search` is empty, `filteredTeams` equals `teams`, so **Export Teams** and **Export All** invoke the same export with the same row set. Both buttons stay visible whenever `teams.length > 0`.

Impact:
- Duplicate controls with no functional difference in the default state
- Extra UI noise on the only teams list page that uses a card grid instead of a paginated table

Recommendation:
Hide **Export All** when `search.trim() === ""`, show it only when a search filter is active (mirroring the two-level export model), or collapse to a single export action until search narrows the list.

**PR comment (line 67):** **Medium:** With no search, Export Teams and Export All export the same data—consider showing one button or differentiating behavior.

---

---
Events export-all wired through table props though parent already owns `filteredRows`

Risk Level: MEDIUM
File Path: src/pages/events/dashboard/index.tsx
Lines: 117-119, 172-178
File Path: src/pages/events/dashboard/components/events-table.tsx
Lines: 125-136

Description:
`filteredRows` is computed in `EventsDashboardPage` and passed into `EventsTable` solely so a child `useEffect` can register `onExportAllReady`. Partners and video library keep filtering inside the table and register handlers there; events already has `filteredRows` at the page level (unlike selection/page state, which lives in the table).

Impact:
- Extra prop surface (`filteredRows`, `onExportAllReady`) on `EventsTable` without rendering use
- Harder to follow than colocating export-all next to `onExportAllClick` in the page (same pattern as `teams/index.tsx`)

Recommendation:
Register export-all in the page and drop the props from the table:

```tsx
// index.tsx
const handleExportAll = useCallback(() => {
  if (filteredRows.length === 0) {
    message.warning("No events to export");
    return;
  }
  exportEventsRowsAsCsv(filteredRows);
}, [filteredRows]);

<EventsHeader onExportAllClick={handleExportAll} ... />
```

Remove `filteredRows`, `onExportAllReady`, and the second `useEffect` from `events-table.tsx`.

**PR comment (line 178):** **Medium:** `filteredRows` is only passed into `EventsTable` for export-all registration—the page already owns that data; consider handling export-all in `index.tsx` like teams and drop the extra props/effect.

---

## Positive notes

- Export-all mirrors existing export patterns (`onExportReady` / `useEffect` cleanup) on partners and video library, including empty-state handlers in `video-library-tab-content.tsx`.
- Tabbed partners/video library only mount the active panel, so `exportAllHandler` tracks the current active/inactive tab correctly.
- Teams CSV utility is colocated under `src/pages/teams/utils/export-teams-csv.ts` with kebab-case naming.
- Empty exports show warnings consistently (`message` / `toast`) before download.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Teams “Export All” ignores search unlike other dashboards | MEDIUM | Open | src/pages/teams/index.tsx | 46-60, 64-68 |
| 2 | Teams export buttons redundant when search is empty | MEDIUM | Open | src/pages/teams/index.tsx | 46-68 |
| 3 | Events export-all wired through table props though parent owns `filteredRows` | MEDIUM | Open | src/pages/events/dashboard/index.tsx | 117-119, 172-178 |
