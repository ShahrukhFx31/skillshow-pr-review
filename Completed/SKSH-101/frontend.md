# Frontend PR Review — skillshow-admin-ui (`SKSH-101`)

**Repo:** skillshow-admin-ui  
**Branch:** `sksh-101`  
**Base:** `main...HEAD`  
**Re-verified:** `5477626a` / `fd132015` (`fix: pr related changes`)  
**Scope:** React performance, hooks, JSX/props, Tailwind/file structure (Critical, High, Medium only)  
**Findings:** 4 total — **3 fixed**, **1 accepted**, **0 open**

---

## Verification summary

| # | Title | Risk | Status | Evidence |
|---|--------|------|--------|----------|
| 1 | Wrong dashboard when `super_admin` is not first role | HIGH | **Fixed** | `resolveDashboardRoleKey()` + `DASHBOARD_ROLE_PRIORITY` with `SuperAdmin` first (`resolve-dashboard-role.utils.ts`, `constant/index.tsx` 114-122); `dashboard/index.tsx` 22 |
| 2 | Placeholder avg rating shown as real in crew table | HIGH | **Fixed** | `avgRating` column removed (`super-admin.utils.tsx` 87-100); `SuperAdminActiveCrewRow` has no rating field (`types.ts` 4-8) |
| 3 | 60s refetch vs 30m staleTime for “live” ops | MEDIUM | **Fixed** | Split queries: charts `staleTime: SUPER_ADMIN_DASHBOARD_CHARTS_STALE_MS` (30m); live `staleTime: SUPER_ADMIN_OPERATIONS_STALE_MS` (60s) + `refetchInterval` (`super-admin.utils.tsx` 130-145, `constants.ts` 59-63) |
| 4 | Duplicate monolithic dashboard fetches | MEDIUM | **Accepted** | Pairs with backend #3 — dual `useQuery` + monolithic API accepted |

---

## Fixed (verified)

### #1 — Dashboard role priority

`Dashboard` uses `resolveDashboardRoleKey(roles)` instead of `roles[0]`. Priority list includes `UserRole.SuperAdmin` before coach/parent/athlete.

### #2 — Avg. rating column removed

`SUPER_ADMIN_ACTIVE_CREW_COLUMNS` is editor + active assignments only; types and mappers match API.

### #3 — StaleTime / polling aligned with live vs charts

- `chartsQuery`: 30m stale, `select` → `charts` only  
- `liveQuery`: 60s stale + 60s `refetchInterval`, `select` → `{ kpis, operations }`  
- Page uses `isChartsFetching` / `isLiveFetching` separately (`super-admin-dashboard-page.tsx`)

---

## Accepted (no change required)

### #4 — Duplicate monolithic dashboard fetches

**Decision (team):** Accept dual `useQuery` calls to `getSuperAdminDashboard` (`chartsQuery` + `liveQuery`). Same trade-off as backend review **#3** (monolithic handler, no chart-only split)—acceptable load for this PR.

**Evidence:** `super-admin.utils.tsx` 130–145 — `chartsQuery` with date params + `liveQuery` with `{}`, separate `staleTime` / `refetchInterval`.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Wrong dashboard when `super_admin` is not first role | HIGH | Fixed | `src/pages/dashboard/index.tsx` | 9, 22 |
| 2 | Placeholder avg rating in crew table | HIGH | Fixed | `src/pages/dashboard/superAdmin/utils/super-admin.utils.tsx` | 87-100 |
| 3 | 60s refetch vs 30m staleTime for live ops | MEDIUM | Fixed | `src/pages/dashboard/superAdmin/utils/super-admin.utils.tsx` | 130-145 |
| 4 | Duplicate monolithic dashboard fetches | MEDIUM | Accepted | `src/pages/dashboard/superAdmin/utils/super-admin.utils.tsx` | 130-145 |

**Positive notes (fix pass):** `resolveDashboardRoleKey` shared utility; crew “View All” uses `CREW_USERS_LIST_PATH`; split loading states for KPIs vs charts vs live tables.

**Skipped (per request):** Pagination/preview limits.

**Not reported:** Ad-hoc palette / Ant `!` overrides (out of scope). Volume chart all-zero → empty state unchanged.
