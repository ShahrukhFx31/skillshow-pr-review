# Frontend PR Review — skillshow-admin-ui (`SKSH-194`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-194-1`  
**Base:** `main...HEAD` @ `1229d8c`  
**Scope:** Crew user work dashboard — summary cards, tabbed edit-request/events tables, Work mode on crew user view/onboarding page  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 4 in scope (0 Critical, 2 High, 2 Medium) — **4 Open**

### Protected modules

| Module | Status |
|--------|--------|
| `use-pagination.ts`, `pagination-bar.tsx`, `use-server-table-controls.ts`, `destructive-action-confirm-modal.tsx`, audit-log stack, `antd.adapter.tsx` | **Not modified** ✅ |

### Files reviewed (13 changed)

`crewUserService.ts`, crew user breadcrumb/onboarding/types, `work/` panel (actions, columns, table, responsive, constants, utils, types).

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| `useServerTableControls` + `extraFilterState` (tab/filters) resets page via `filterKey` | ✅ |
| `applyServerSort`, `PaginationBar`, hidden Ant pagination, `TableEmptyState` | ✅ (matches crew-users dashboard pattern) |
| `queryKey` includes tab, filters, search, sort, pagination | ✅ |
| Edit-request status mapped UI → backend via `mapUiStatusFilterToBackendStatus` | ✅ |
| Reuses event column keys, tags, `mapEventListApiItemToTableRow` | ✅ |
| Work mode entered via `location.state` on **view** URL; `crewUserRoutes.work` unused | ⚠️ See #1 |
| Events search placeholder claims start-date search; API is regex text fields only | ⚠️ See #2 |
| Edit-request search placeholder says "requester"; search targets athlete user | ⚠️ See #3 |
| Event table columns largely copy-pasted from events dashboard | ⚠️ See #4 |

---

## GitHub comments (Open findings)

### 1. `src/pages/management/crew-users/onboarding/index.tsx` line 117

**PR comment (line 117):** **High (Contract / KISS):** Work mode is driven by `location.state.mode` while the URL stays on `/view/:userId`. Refresh or a shared link drops state and renders the profile form instead of the work dashboard. Navigate to `crewUserRoutes.work(userId)` (and ensure that route renders this page or a thin wrapper) — `pathname.includes("/work/")` already anticipates a real work path.

### 2. `src/pages/management/crew-users/work/constants.ts` line 36

**PR comment (line 36):** **High (Contract):** Events tab placeholder promises "start date" search, but `listCrewUserWorkEvents` only regex-matches `eventName`, `partnerSource`, `facility`, `city`, and `state`. Update the copy or add backend date parsing for event start dates (same pattern as edit-request date search).

### 3. `src/pages/management/crew-users/work/constants.ts` line 32

**PR comment (line 32):** **Medium (Contract):** Placeholder says "requester" but the API searches edit-request title and athlete username/email (`resolveCrewWorkEditRequestSearchOpts`). Change to "user" or "athlete" unless backend adds requester (`requestedBy`) search.

### 4. `src/pages/management/crew-users/work/components/crew-user-work-columns.tsx` line 187

**PR comment (line 187):** **Medium (DRY):** `CREW_USER_WORK_EVENT_BASE_COLUMNS` duplicates the events dashboard column definitions (IDs, tags, date formatters). Consider importing or extracting a shared `buildEventListDataColumns()` from the events feature to avoid drift when event columns change.

---

## Findings

---
Work dashboard mode is not URL-addressable (lost on refresh)

Risk Level: HIGH  
File Path: src/pages/management/crew-users/onboarding/index.tsx  
Lines: 27-31, 117-121

Description:
**Contract / KISS.** Work mode is activated when `location.state.mode === CrewUserFormMode.Work` or `pathname.includes("/work/")`. The Work button navigates to the **view** path (`crewUserRoutes.view`) and only sets `state: { mode: CrewUserFormMode.Work }`. `crewUserRoutes.work(userId)` exists in breadcrumb helpers but is never used, and `work/index.tsx` has no default page export for a standalone `/work/:userId` route.

Impact:
- Browser refresh or opening a bookmarked view URL shows the crew user form instead of the work dashboard
- Shared admin links cannot deep-link to Work
- Breadcrumb trail remains "View Crew User" while Work content is shown

Recommendation:
Navigate with `navigateCrewUser(navigate, crewUserRoutes.work(userId))` (or equivalent) so the URL is `/user-management/crew-users/work/:userId`. Ensure the permission route loads a page component that renders `CrewUserWorkPanel` (default export on `work/index.tsx` or reuse onboarding with path-based mode). Drop reliance on ephemeral `location.state.mode` for primary navigation.

```typescript
onClick={() => navigateCrewUser(navigate, crewUserRoutes.work(userId))}
```

**PR comment (line 117):** **High:** Use the `/work/:userId` route for Work navigation so refresh and deep links keep the work dashboard.

---

---
Events tab search placeholder promises unsupported start-date search

Risk Level: HIGH  
File Path: src/pages/management/crew-users/work/constants.ts  
Lines: 34-37

Description:
**Contract.** `CREW_USER_WORK_SEARCH.events.placeholder` tells admins they can search by "start date". The backend crew-work events list applies `buildRegexOrMatch` only on `eventName`, `partnerSource`, `facility`, `city`, and `state` — there is no date parsing (unlike edit-request search, which uses `parseEventDatePayload` server-side).

Impact:
- Admins enter dates expecting filtered results and see empty or unrelated rows
- Cross-stack contract mismatch between UI copy and API behavior

Recommendation:
Either update the placeholder to match supported fields (e.g. "Search by event name, location, or partner") or add date-aware search to `listEventsPaginatedByCrewUser` / service layer consistent with edit-request work search.

**PR comment (line 36):** **High (Contract):** Remove "start date" from the placeholder or implement date search on the events work list endpoint.

---

---
Edit-request search placeholder mislabels athlete user search as "requester"

Risk Level: MEDIUM  
File Path: src/pages/management/crew-users/work/constants.ts  
Lines: 31-33

Description:
**Contract.** Placeholder: "Search by date, title, or requester". Backend `resolveCrewWorkEditRequestSearchOpts` matches title regex and resolves athlete `userId` via username/email — not the `requestedBy` (`self` / `parent`) field shown in the Requester column.

Impact:
- Admins search for "Parent" or a parent name and get no useful results
- Column label "Requester" vs search behavior causes confusion

Recommendation:
Change placeholder to "Search by date, title, request ID, or user" (EDR prefix is supported server-side), or extend backend search to cover `requestedBy` / submitter actor if product requires it.

**PR comment (line 32):** **Medium:** Align placeholder with athlete user search, not the Requester column semantics.

---

---
Duplicated events dashboard column definitions in crew work table

Risk Level: MEDIUM  
File Path: src/pages/management/crew-users/work/components/crew-user-work-columns.tsx  
Lines: 187-262

Description:
**DRY.** `CREW_USER_WORK_EVENT_BASE_COLUMNS` mirrors events dashboard columns: `EVENT_COLUMN_KEYS`, `EventTypeTableTag`, `EventLifecycleStatusTableTag`, `parseEventDateForUi`, `formatEventIdForDisplay`, etc. The crew work table will drift when event list columns are updated elsewhere.

Impact:
- Inconsistent event table UX across admin surfaces over time
- Duplicate maintenance when event DTO fields or display rules change

Recommendation:
Extract shared event list data columns (without action column) from `src/pages/events/dashboard/components/event-columns.tsx` or a colocated `event-list-columns.ts` and import into crew work columns.

**PR comment (line 187):** **Medium (DRY):** Reuse shared event list column builders instead of copying dashboard column config.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Work dashboard mode is not URL-addressable (lost on refresh) | HIGH | Open | src/pages/management/crew-users/onboarding/index.tsx | 27-31, 117-121 |
| 2 | Events tab search placeholder promises unsupported start-date search | HIGH | Open | src/pages/management/crew-users/work/constants.ts | 34-37 |
| 3 | Edit-request search placeholder mislabels athlete user search as "requester" | MEDIUM | Open | src/pages/management/crew-users/work/constants.ts | 31-33 |
| 4 | Duplicated events dashboard column definitions in crew work table | MEDIUM | Open | src/pages/management/crew-users/work/components/crew-user-work-columns.tsx | 187-262 |

### Positive notes

- `useServerTableControls` correctly includes `activeTab` and filters in `extraFilterState` so tab/filter changes reset pagination.
- Edit-request status filter maps UI enums to backend via existing `editRequestService` helpers.
- `DestructiveActionConfirmModal` / audit-log protected modules untouched.
- Responsive cards reuse column `render` functions via `buildCrewUserWorkEditRequestDataColumns` — good DRY within the feature.
- Tab switch clears search/filters and resets sort to shared default `assignedAt`.

**Merge readiness:** Blocked — 2 open High findings (#1 refresh/deep-link, #2 search contract); address or explicitly accept before merge.
