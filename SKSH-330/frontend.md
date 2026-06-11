# Frontend PR Review — skillshow-admin-ui (`SKSH-330`)

**Repo:** skillshow-admin-ui — `https://github.com/fx31labs-mvp/skillshow-admin-ui.git`  
**Branch:** `sksh-330`  
**Base:** `main...HEAD` @ `13166ea0`  
**Initial review:** 2026-06-11  
**Scope:** Admin Social Platforms management page, global platform availability gating across connect/distribute/share flows, enable/disable confirmation modal (Critical / High / Medium)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [backend.md](./backend.md), [orchestrator.md](./orchestrator.md)

**Findings:** 5 (0 Critical, 3 High, 2 Medium) — **5 Open**

### Protected modules

| Module | Status |
|--------|--------|
| `pagination-bar.tsx`, `use-pagination.ts`, `use-server-table-controls.ts`, `table-sort.ts`, `destructive-action-confirm-modal.tsx`, `antd.adapter.tsx`, audit-log components | **Not modified** |

### Files reviewed

| File | Change |
|------|--------|
| `src/api/apiClient.ts` | Show error toast on non-success body for all HTTP methods |
| `src/api/services/socialPlatformService.ts` | List / update social platform APIs |
| `src/components/action-confirm-modal.tsx` | New enable/disable confirmation modal |
| `src/hooks/useSocialPlatformAvailability.ts` | Shared availability query + `isPlatformActive` |
| `src/pages/dashboard/components/ConnectSocialModal.tsx` | Filter providers by global availability |
| `src/pages/management/social-platforms/**` | New admin management page, table, columns, sort utils |
| `src/pages/user/account/share-account-tab.tsx` | Gate connect by platform availability |
| `src/pages/user/account/share-account/components/SocialAccountsSection.tsx` | Hide inactive platforms; show "Unavailable" for stored connections |
| `src/pages/videos/components/ConnectedPlatformSelector.tsx` | Filter upload/distribute platform picker |
| `src/pages/videos/details/index.tsx` | Wire availability into video detail mappers |
| `src/pages/videos/details/mappers.ts` | Filter inactive platforms from platform rows / unconnected list |
| `src/pages/videos/utils/distribute-eligibility.utils.ts` | Exclude inactive platforms from distribute eligibility |
| `src/router/routes/modules/management.tsx` | Register social-platforms route (module mode) |
| `src/utils/social-platform-availability.utils.ts` | Active vendor set helpers |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| `useSocialPlatformAvailability` reused across connect modal, share account, upload selector, video details | ✅ DRY |
| Availability cache invalidated after admin status toggle | ✅ |
| `mapPlatforms` / `getEligibleDistributePlatforms` / `ConnectedPlatformSelector` all accept availability filter | ✅ Global consistency |
| Published vendor logs still shown when platform later disabled | ✅ Correct |
| Social platforms admin list uses hand-rolled pagination/sort instead of frozen table hooks | ❌ Contract — see #1 |
| `TeamTablePaginationBar` + `PAGE_SIZE_OPTIONS` includes 100 while backend max is 50 | ❌ Contract — see #2 |
| `isPlatformActive` returns `true` while availability query is loading | ❌ Behavior — see #3 |
| `ActionConfirmModal` largely duplicates `DestructiveActionConfirmModal` shell | ❌ DRY — see #4 |
| Protected table/pagination modules untouched | ✅ |

### Positive notes

- **Feature wiring:** Availability enforcement is applied consistently at connect entry points, distribute eligibility, upload picker, and video detail mappers — backend `assertVendorsActive` is complemented on the client.
- **UX:** Connected-but-disabled platforms remain visible in Share Account with an "Unavailable" badge instead of disappearing silently.
- **Cache invalidation:** Status toggle mutation invalidates both the admin list and `SOCIAL_PLATFORM_AVAILABILITY_QUERY_KEY`.
- **Confirmation flow:** Enable/disable uses a confirmation step with contextual copy before PATCH.

---

## GitHub comments

### 1. `src/pages/management/social-platforms/index.tsx` line 34

**PR comment (line 34):** **High (Contract):** This page hand-rolls `page` / `pageSize` / sort state and a custom `socialPlatformTableSort` helper instead of the frozen server-table stack (`usePagination` or `useServerTableControls`, `applyServerSort`, `PaginationBar`). Please align with sibling management list pages (e.g. crew-users) so pagination reset, sort wiring, and bar markup stay consistent.

### 2. `src/pages/management/social-platforms/index.tsx` line 78

**PR comment (line 78):** **High (Contract):** `handleTableChange` uses bespoke `resolveSocialPlatformTableSortChange` instead of `applyServerSort` from `table-sort.ts`. Wire sort through the shared helper and map `sortBy` → API `sortField` in the query function only.

### 3. `src/pages/management/social-platforms/components/social-platforms-table.tsx` line 47

**PR comment (line 47):** **High (Contract):** `TeamTablePaginationBar` exposes page sizes up to 100, but the social-platforms API caps `pageSize` at 50. Restrict options for this table (max 50) or clamp before the request so the UI label matches what the backend returns.

### 4. `src/hooks/useSocialPlatformAvailability.ts` line 25

**PR comment (line 25):** **High:** `isVendorActive` returns `true` when `query.data` is still undefined, so disabled platforms briefly appear connectable/distributable until the list loads. Return `false` while loading or gate connect/distribute actions on `isLoading` to avoid a flash of incorrect availability.

### 5. `src/components/action-confirm-modal.tsx` line 66

**PR comment (line 66):** **Medium (DRY):** `ActionConfirmModal` duplicates most of the Modal/Drawer structure already in `DestructiveActionConfirmModal`. Consider extracting a shared confirm shell (tone/icon variants) so both components reuse one implementation.

### 6. `src/pages/management/social-platforms/components/social-platforms-table.tsx` line 38

**PR comment (line 38):** **Medium (Contract):** The table passes `locale={{ emptyText }}` without defaulting to `TableEmptyState`. Use the shared empty-state component for consistency with other server-driven tables.

---

## Findings

---
Social platforms list bypasses frozen server-table controls

Risk Level: HIGH  
**Status:** Open  
File Path: src/pages/management/social-platforms/index.tsx  
Lines: 34-37, 78-84

Description:
**Contract / DRY / Global consistency.** The new Social Platforms admin page manages server-driven pagination and sort with local `useState`, a bespoke `socialPlatformTableSort.ts` module, and `TeamTablePaginationBar`. Sibling management list pages (crew-users, skillshow-users, partners) use `useServerTableControls` + `usePagination` + `PaginationBar` + `applyServerSort` from the frozen modules.

Impact:
- Sort/pagination reset behavior can drift from established list pages.
- Duplicated sort parsing (`resolveSocialPlatformTableSortChange`) will diverge from `applyServerSort` over time.
- Future table-contract fixes must be applied twice.

Recommendation:
Refactor `SocialPlatformsPage` to use `usePagination` (or `useServerTableControls` with `extraFilterState: {}`), wire table `onChange` through `applyServerSort`, and render `PaginationBar` via `usePagination`'s `hidden`/`bar` pattern. Map `sortBy` → API `sortField` in the query function only.

**PR comment (`index.tsx` line 34):** **High (Contract):** Hand-rolled pagination/sort state — please use `usePagination` / `useServerTableControls` + `PaginationBar` like other management list pages.

**PR comment (`index.tsx` line 78):** **High (Contract):** Replace `resolveSocialPlatformTableSortChange` with `applyServerSort` from `table-sort.ts`.

---

---
Pagination bar allows pageSize 100 but API max is 50

Risk Level: HIGH  
**Status:** Open  
File Path: src/pages/management/social-platforms/components/social-platforms-table.tsx  
Lines: 47-53

Description:
**Contract / Global consistency.** `TeamTablePaginationBar` uses global `PAGE_SIZE_OPTIONS` (`[5, 10, 20, 50, 100]`). Backend social-platforms list clamps `pageSize` to `SOCIAL_PLATFORM_LIST_MAX_PAGE_SIZE` (50). Selecting 100 sends `pageSize: 100`; the server silently serves 50 rows while the UI still shows "100 rows per page".

Impact:
- Mismatched rows-per-page label vs actual data on the admin page.
- Cross-stack contract violation between frontend pagination controls and backend bounds.

Recommendation:
Pass a capped option list (e.g. `PAGE_SIZE_OPTIONS.filter((n) => n <= 50)`) to this table's pagination bar, or define `SOCIAL_PLATFORMS_PAGE_SIZE_OPTIONS` in feature constants aligned with the backend max.

**PR comment (`social-platforms-table.tsx` line 47):** **High (Contract):** Pagination offers 100 rows/page but the API max is 50 — cap page-size options or clamp the request param.

---

---
Disabled platforms treated as active while availability query is loading

Risk Level: HIGH  
**Status:** Open  
File Path: src/hooks/useSocialPlatformAvailability.ts  
Lines: 23-26

Description:
**Behavior / Global consistency.** `isVendorActive` returns `true` when `query.data` is undefined (initial load). Every consumer (`ConnectSocialModal`, `ConnectedPlatformSelector`, `share-account-tab`, video details) therefore shows all platforms as available until the first `/v1/social-platforms` response arrives.

Impact:
- Brief window where admin-disabled platforms appear in connect/distribute pickers.
- User may start OAuth or select a platform that the backend will reject once data loads (backend enforces, but UX is misleading).

Recommendation:
Return `false` (or `isPlatformActive` → false) while `query.isLoading && !query.data`, and disable connect/distribute affordances until the query settles. Alternatively expose `isReady` from the hook and gate UI on it.

**PR comment (`useSocialPlatformAvailability.ts` line 25):** **High:** `if (!query.data) return true` treats all platforms as active while loading — return `false` or gate UI on `isLoading`.

---

---
ActionConfirmModal duplicates DestructiveActionConfirmModal structure

Risk Level: MEDIUM  
**Status:** Open  
File Path: src/components/action-confirm-modal.tsx  
Lines: 42-138

Description:
**DRY.** `ActionConfirmModal` reimplements the same centered icon body, mobile `Drawer` / desktop `Modal` split, loading-guarded close, and footer button layout already present in `DestructiveActionConfirmModal` (protected module — do not edit in this ticket).

Impact:
- Two confirm-modal implementations to maintain; styling/behavior fixes must be duplicated.
- Risk of inconsistent loading/mask-close behavior between delete and enable/disable flows.

Recommendation:
Extract a shared internal `ConfirmActionModalShell` (icon, tone styles, responsive layout) under `src/components/`, then thin-wrap it from `ActionConfirmModal` (tone variants) and eventually from `DestructiveActionConfirmModal` in a dedicated refactor ticket.

**PR comment (`action-confirm-modal.tsx` line 66):** **Medium (DRY):** This Modal/Drawer shell duplicates `DestructiveActionConfirmModal` — extract a shared confirm body/layout component.

---

---
Social platforms table missing TableEmptyState

Risk Level: MEDIUM  
**Status:** Open  
File Path: src/pages/management/social-platforms/components/social-platforms-table.tsx  
Lines: 38

Description:
**Contract.** Server-driven tables in this codebase use `TableEmptyState` (or `TABLE_EMPTY_DESCRIPTION`) for `locale.emptyText`. This table passes `locale={{ emptyText }}` with no default, so when the page omits `emptyText` Ant Design's generic empty renders.

Impact:
- Inconsistent empty UI vs sibling management tables in the same PR area.

Recommendation:
Import `TableEmptyState` and set `locale={{ emptyText: emptyText ?? <TableEmptyState /> }}` (or pass `TABLE_EMPTY_DESCRIPTION` from the page).

**PR comment (`social-platforms-table.tsx` line 38):** **Medium (Contract):** Default `locale.emptyText` to `TableEmptyState` instead of relying on Ant's generic empty.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Social platforms list bypasses frozen server-table controls | HIGH | Open | src/pages/management/social-platforms/index.tsx | 34-37, 78-84 |
| 2 | Pagination bar allows pageSize 100 but API max is 50 | HIGH | Open | src/pages/management/social-platforms/components/social-platforms-table.tsx | 47-53 |
| 3 | Disabled platforms treated as active while availability query is loading | HIGH | Open | src/hooks/useSocialPlatformAvailability.ts | 23-26 |
| 4 | ActionConfirmModal duplicates DestructiveActionConfirmModal structure | MEDIUM | Open | src/components/action-confirm-modal.tsx | 42-138 |
| 5 | Social platforms table missing TableEmptyState | MEDIUM | Open | src/pages/management/social-platforms/components/social-platforms-table.tsx | 38 |

**Merge readiness:** **Not merge-ready** — 3 open High findings (server-table contract, pageSize bounds, loading availability flash). Address High items before merge; Medium items are recommended follow-ups.
