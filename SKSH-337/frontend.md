# Frontend PR Review — skillshow-admin-ui (`SKSH-337`)

**Repo:** skillshow-admin-ui — `https://github.com/fx31labs-mvp/skillshow-admin-ui.git`  
**Branch:** `SKSH-337`  
**Base:** `main...HEAD` @ `517b3a87`  
**Initial review:** 2026-06-11  
**Scope:** Crew dashboard "Recent Assigned Events" section — server-paginated table, API hook, realtime invalidation; performance-reviews page-size tweak (Critical / High / Medium)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [backend.md](./backend.md)

**Findings:** 2 (0 Critical, 2 High) — **2 Open**

### Protected modules

| Module | Status |
|--------|--------|
| `pagination-bar.tsx`, `use-pagination.ts`, `use-server-table-controls.ts`, `table-sort.ts`, `destructive-action-confirm-modal.tsx`, `antd.adapter.tsx`, audit-log components | **Not modified** |

### Files reviewed

| File | Change |
|------|--------|
| `src/api/services/crewDashboardService.ts` | `getCrewAssignedEvents` |
| `src/api/types/crew-dashboard.types.ts` | Assigned-events DTOs |
| `src/constants/queryKeys.ts` | `crewAssignedEventsQueryKey` |
| `src/pages/dashboard/crew/dashboard/components/assigned-events-*` | Section, table, columns, responsive cards |
| `src/pages/dashboard/crew/dashboard/components/crew-dashboard-truncated-name.tsx` | Shared truncated name + tooltip |
| `src/pages/dashboard/crew/dashboard/components/edit-requests-columns.tsx` | Reuse `CrewDashboardTruncatedName` |
| `src/pages/dashboard/crew/dashboard/hooks/use-crew-assigned-events.ts` | React Query hook |
| `src/pages/dashboard/crew/dashboard/index.tsx` | Wire `AssignedEventsSection` |
| `src/pages/dashboard/crew/dashboard/utils/*` | Mappers + name preview util |
| `src/utils/edit-request-realtime.utils.ts` | Invalidate/refetch assigned-events on socket events |
| `src/pages/user/account/general/crew/**` | Performance reviews page size 5 → 10 (tangential) |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| `CrewDashboardTruncatedName` + `crewDashboardNamePreview` reused in edit-requests + assigned-events | ✅ DRY |
| `EventLifecycleStatusTableTag` for status column | ✅ Reuse |
| `mapCrewAssignedEvents` colocated with other crew dashboard mappers | ✅ |
| Realtime utils invalidate/refetch `crewAssignedEventsQueryKey` | ✅ |
| `usePagination` in section but `hidden`/`bar` re-implemented in table | ❌ Contract — see #1 |
| `PaginationBar` page sizes up to 100 vs backend service cap 25 | ❌ Cross-stack — see #2 |
| Protected table/pagination modules untouched | ✅ |

### Positive notes

- **Feature completeness:** Assigned events section mirrors edit-requests layout (Card + desktop table + mobile responsive cards + `PaginationBar`).
- **Server pagination:** `useCrewAssignedEvents` passes `page`/`limit` to the API and includes both in `queryKey` — correct cache segmentation.
- **Realtime:** Socket-driven `invalidateCrewDashboardQueries` / `refetchCrewDashboardQueries` now cover assigned events alongside KPIs and performance reviews.

---

## GitHub comments

### 1. `src/pages/dashboard/crew/dashboard/components/assigned-events-section.tsx` line 16

**PR comment (line 16):** **High (Contract):** `usePagination` is called with hardcoded `total: 0`, so only page state is used while `AssignedEventsTable` re-implements hidden Ant pagination + `PaginationBar` props. Derive `bar`/`hidden` from `page`, `pageSize`, API `total`, and `setPage`/`setPageSize` in the section (same pattern as `events-table.tsx`) and pass them to the table instead of duplicating the hidden-pagination config.

### 2. `src/pages/dashboard/crew/dashboard/components/assigned-events-table.tsx` line 56

**PR comment (line 56):** **High (Contract):** `PaginationBar` exposes page sizes up to 100, but the crew assigned-events API service still caps `limit` at 25 (`PAGINATION.MAX_LIMIT`). Until backend caps align with validation, restrict page-size options for this table (max 25) or clamp `limit` before the request so row count matches the selected page size.

---

## Findings

---
Assigned events pagination bypasses usePagination bar/hidden

Risk Level: HIGH  
**Status:** Open  
File Path: skillshow-admin-ui/src/pages/dashboard/crew/dashboard/components/assigned-events-section.tsx  
Lines: 16-43

Description:
**Contract / DRY.** `AssignedEventsSection` calls `usePagination` with a hardcoded `total` of `0`, consuming only `page` / `pageSize` / setters. `AssignedEventsTable` then re-implements the frozen pagination contract — hidden Ant `pagination` config (lines 22–33) and explicit `PaginationBar` props (lines 55–63) — instead of using `usePagination`'s `hidden` and `bar` outputs.

```16:43:skillshow-admin-ui/src/pages/dashboard/crew/dashboard/components/assigned-events-section.tsx
	const { page, pageSize, setPage, setPageSize } = usePagination(
		CREW_ASSIGNED_EVENTS_TABLE_FILTER_KEY,
		0,
		CREW_ASSIGNED_EVENTS_PAGE_SIZE,
	);
	const { isPending, rows, total } = useCrewAssignedEvents(page, pageSize);
	// ...
			<AssignedEventsTable
				isPending={isPending}
				onPageChange={setPage}
				onPageSizeChange={setPageSize}
				page={page}
				pageSize={pageSize}
				rows={rows}
				total={total}
			/>
```

Sibling server lists (e.g. `events-table.tsx`) colocate `bar`/`hidden` next to API `total` and pass them down — this PR introduces a third variant for the same dashboard feature family.

Impact:
- Duplicated hidden-pagination markup will drift from `usePagination` / `PaginationBar` behavior (e.g. `showLessItems`, zero-total handling).
- Partial `usePagination` usage adds indirection without benefiting from the frozen hook contract.

Recommendation:
Colocate pagination UI with API `total` in the section (mirror `events-table.tsx` lines 35–37), then pass `hidden` and `bar` into `AssignedEventsTable`:

```tsx
const { page, pageSize, setPage, setPageSize } = usePagination(
  CREW_ASSIGNED_EVENTS_TABLE_FILTER_KEY,
  0,
  CREW_ASSIGNED_EVENTS_PAGE_SIZE,
);
const { isPending, rows, total } = useCrewAssignedEvents(page, pageSize);

const hidden =
  total === 0
    ? false
    : { current: page, onChange: setPage, pageSize, showLessItems: true, showSizeChanger: false, style: { display: "none" }, total };
const bar = { page, pageSize, setPage, setPageSize, total };
```

Alternatively, merge page state + query + `bar`/`hidden` into a colocated `useAssignedEventsTable` hook and delete the duplicate hidden-pagination block from `AssignedEventsTable`.

**PR comment (`assigned-events-section.tsx` line 16):** **High (Contract):** `usePagination` is called with hardcoded `total: 0`, so only page state is used while `AssignedEventsTable` re-implements hidden Ant pagination + `PaginationBar` props. Derive `bar`/`hidden` from `page`, `pageSize`, API `total`, and `setPage`/`setPageSize` in the section (same pattern as `events-table.tsx`) and pass them to the table instead of duplicating the hidden-pagination config.

---

---
PaginationBar page sizes exceed backend crew limit cap

Risk Level: HIGH  
**Status:** Open  
File Path: skillshow-admin-ui/src/pages/dashboard/crew/dashboard/components/assigned-events-table.tsx  
Lines: 55-63

Description:
**Contract (cross-stack).** `PaginationBar` renders `PAGE_SIZE_OPTIONS` `[5, 10, 20, 50, 100]`. The crew assigned-events API validates `limit` up to 100 (this PR) but the service layer still clamps to `PAGINATION.MAX_LIMIT` (**25**) on the backend. When a user selects 50 or 100 rows per page, the UI label does not match the number of rows returned.

Impact:
- Crew users see fewer rows than the selected page size with no error feedback.
- `totalPages` / page navigation can appear correct while row density is wrong.

Recommendation:
Until backend service caps align with validation (see [backend.md](./backend.md) #2), either:
1. Pass a restricted `pageSizeOptions` prop to `PaginationBar` (max 25) for this table, or
2. Clamp `limit` in `getCrewAssignedEvents` / `useCrewAssignedEvents` before the request and sync displayed `pageSize` to the clamped value.

**PR comment (`assigned-events-table.tsx` line 56):** **High (Contract):** `PaginationBar` exposes page sizes up to 100, but the crew assigned-events API service still caps `limit` at 25 (`PAGINATION.MAX_LIMIT`). Until backend caps align with validation, restrict page-size options for this table (max 25) or clamp `limit` before the request so row count matches the selected page size.

---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Assigned events pagination bypasses usePagination bar/hidden | HIGH | Open | skillshow-admin-ui/src/pages/dashboard/crew/dashboard/components/assigned-events-section.tsx | 16-43 | 16 |
| 2 | PaginationBar page sizes exceed backend crew limit cap | HIGH | Open | skillshow-admin-ui/src/pages/dashboard/crew/dashboard/components/assigned-events-table.tsx | 55-63 | 56 |

**Merge readiness:** **Not merge-ready** — 2 open High findings (pagination contract + cross-stack page-size cap). Fix before merge.
