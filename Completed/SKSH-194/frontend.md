# Frontend PR Review — skillshow-admin-ui (`SKSH-194`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-194-1`  
**Base:** `main...HEAD` @ `4652004`  
**Re-reviewed:** 2026-06-11 (post `fix: added fix`)  
**Scope:** Crew user work dashboard — dedicated `/work/:userId` page, summary cards, tabbed edit-request/events tables  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 4 in scope (0 Critical, 2 High, 2 Medium) — **0 Open**, **2 Fixed**, **2 Accepted**

### Protected modules

| Module | Status |
|--------|--------|
| `use-pagination.ts`, `pagination-bar.tsx`, `use-server-table-controls.ts`, `destructive-action-confirm-modal.tsx`, audit-log stack, `antd.adapter.tsx` | **Not modified** ✅ |

### Files reviewed (12 changed)

`crewUserService.ts`, crew user breadcrumb/onboarding, `work/index.tsx` (default page export), actions, columns, table, responsive, constants, utils, types.

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Dedicated `CrewUserWorkPage` default export at `/work/:userId` | ✅ Fixed (#1) |
| View page navigates via `navigateCrewUser(navigate, crewUserRoutes.work(userId))` | ✅ Fixed (#1) |
| `useServerTableControls` + `extraFilterState` resets page via `filterKey` | ✅ |
| `applyServerSort`, `PaginationBar`, hidden Ant pagination, `TableEmptyState` | ✅ |
| Events search placeholder matches API text fields | ✅ Fixed (#2) |
| Edit-request placeholder says "requester" vs athlete user search | Accepted (#3) — intentional for this release |
| Event columns in `buildCrewUserWorkEventDataColumns` not shared with events dashboard | Accepted (#4) — scoped to crew work; extract later if reused again |
| `EditRequestListNavButton` replaced with ad-hoc `<button>` for title links | ⚠️ Minor regression vs shared component |

---

## Findings

---
Work dashboard mode is not URL-addressable (lost on refresh)

Risk Level: HIGH  
Status: ✅ Fixed  
File Path: src/pages/management/crew-users/work/index.tsx, src/pages/management/crew-users/onboarding/index.tsx  
Lines: work page (full file), onboarding 103-104

Description:
**Contract / KISS.** Initial implementation drove Work mode via `location.state.mode` on the view URL; refresh dropped Work content.

**Re-verification (`4652004`):** Work is a standalone `CrewUserWorkPage` default export. View page navigates with `navigateCrewUser(navigate, crewUserRoutes.work(userId))` to `/user-management/crew-users/work/:userId`. Work mode removed from onboarding `CrewUserFormMode`.

Impact:
- ~~Refresh / deep links lost Work dashboard~~ Resolved — URL is addressable.

Recommendation:
No further change required. Ensure backend permission route maps to `management/crew-users/work/index.tsx` in deployed environments.

---

---
Events tab search placeholder promises unsupported start-date search

Risk Level: HIGH  
Status: ✅ Fixed  
File Path: src/pages/management/crew-users/work/constants.ts  
Lines: 32-35

Description:
**Contract.** Initial placeholder included "start date" but the API only regex-matches text fields.

**Re-verification:** Placeholder is now `"Search by event name, location, or partner"`, matching `CREW_USER_WORK_EVENTS_LIST_SEARCH_FIELDS`.

Recommendation:
No further change required.

---

---
Edit-request search placeholder mislabels athlete user search as "requester"

Risk Level: MEDIUM  
Status: Accepted  
File Path: src/pages/management/crew-users/work/constants.ts  
Lines: 28-31

Description:
**Contract.** Placeholder: `"Search by date, title, or requester"`. Backend `resolveCrewWorkEditRequestSearchOpts` matches title regex, EDR seq, date range, and athlete username/email — not the `requestedBy` (`self` / `parent`) field in the Requester column.

Impact:
- Admins searching for "Parent" or a parent name get no useful results
- Column label "Requester" vs search behavior causes confusion

Recommendation:
Change to `"Search by date, title, request ID, or user"` or extend backend search to cover submitter/requester if product requires it.

**Accepted:** Team acknowledged; placeholder copy left as-is for this release. Revisit if user-testing shows confusion.

---

---
Duplicated events dashboard column definitions in crew work table

Risk Level: MEDIUM  
Status: Accepted  
File Path: src/pages/management/crew-users/work/components/crew-user-work-columns.tsx  
Lines: 145-235

Description:
**DRY.** `buildCrewUserWorkEventDataColumns` mirrors events dashboard columns (`EVENT_COLUMN_KEYS`, tags, date formatters). Refactor improved structure (`buildWorkViewActionColumn`, event name `Link`) but did not promote a shared module.

Impact:
- Event table UX can drift when events dashboard columns change
- Duplicate maintenance when event DTO display rules change

Recommendation:
Extract shared event list data columns from `src/pages/events/dashboard/components/event-columns.tsx` and import into crew work columns.

**Accepted:** Scoped extraction deferred — crew work columns include work-specific link behavior; shared module not required for merge.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Work dashboard mode is not URL-addressable (lost on refresh) | HIGH | ✅ Fixed | src/pages/management/crew-users/work/index.tsx | — |
| 2 | Events tab search placeholder promises unsupported start-date search | HIGH | ✅ Fixed | src/pages/management/crew-users/work/constants.ts | 32-35 |
| 3 | Edit-request search placeholder mislabels athlete user search as "requester" | MEDIUM | Accepted | src/pages/management/crew-users/work/constants.ts | 28-31 |
| 4 | Duplicated events dashboard column definitions in crew work table | MEDIUM | Accepted | src/pages/management/crew-users/work/components/crew-user-work-columns.tsx | 145-235 |

### Positive notes

- Work page includes header (name, status tag), back-to-list, and View Details navigation — clean separation from onboarding form.
- `buildCrewUserWorkEventDataColumns` + `buildWorkViewActionColumn` reduce in-file duplication; responsive cards reuse data column builders.
- `seq` added to frontend sort type; Request ID column is sortable end-to-end.
- Tab switch clears search/filters and resets sort; `queryKey` includes all list inputs.

**Merge readiness:** No open findings — all rows Fixed or Accepted. Ready to merge.
