# Frontend PR Review — skillshow-admin-ui (`SKSH-330`)

**Repo:** skillshow-admin-ui — `https://github.com/fx31labs-mvp/skillshow-admin-ui.git`  
**Branch:** `sksh-330`  
**Base:** `main...HEAD` @ `ad3b8a59`  
**Initial review:** 2026-06-11  
**Re-reviewed:** 2026-06-11 (`ad3b8a59` — feedback: server-table contract, availability loading, confirm modal DRY)  
**Scope:** Admin Social Platforms management, global platform availability gating, enable/disable confirmation (Critical / High / Medium)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [backend.md](./backend.md), [orchestrator.md](./orchestrator.md)

**Findings:** 6 (0 Critical, 3 High, 3 Medium) — **0 Open**, **5 Fixed**, **1 Accepted**

### Protected modules

| Module | Status |
|--------|--------|
| `pagination-bar.tsx`, `use-pagination.ts`, `use-server-table-controls.ts`, `table-sort.ts`, `destructive-action-confirm-modal.tsx`, `antd.adapter.tsx`, audit-log components | **Not modified** |

### Files reviewed

| File | Change |
|------|--------|
| `src/api/apiClient.ts` | Show error toast on non-success body for all HTTP methods |
| `src/api/services/socialPlatformService.ts` | List / update social platform APIs |
| `src/components/action-confirm-modal.tsx` | Thin wrapper over shared shell |
| `src/components/confirm-action-modal-shell.tsx` | **New** shared Modal/Drawer confirm layout |
| `src/hooks/useSocialPlatformAvailability.ts` | Availability query + `isPlatformActive` |
| `src/pages/management/social-platforms/**` | Admin page, table, columns, constants |
| `src/pages/dashboard/components/ConnectSocialModal.tsx` | Filter providers by availability |
| `src/pages/user/account/share-account*` | Gate connect; show "Unavailable" badge |
| `src/pages/videos/components/ConnectedPlatformSelector.tsx` | Filter upload/distribute picker |
| `src/pages/videos/details/*` | Wire availability into mappers |
| `src/pages/videos/utils/distribute-eligibility.utils.ts` | Exclude inactive platforms |
| `src/router/routes/modules/management.tsx` | Register social-platforms route |
| `src/utils/social-platform-availability.utils.ts` | Active vendor set helpers |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| `useSocialPlatformAvailability` reused across connect, share, upload, video details | ✅ DRY |
| Availability cache invalidated after admin toggle | ✅ |
| `mapPlatforms` / distribute eligibility / selectors all filter by availability | ✅ Global consistency |
| Social platforms list uses `usePagination`, `PaginationBar`, `applyServerSort`, `TableEmptyState` | ✅ Fixed |
| `pageSize` capped to `SOCIAL_PLATFORM_LIST_MAX_PAGE_SIZE` (50) | ✅ Fixed |
| `isVendorActive` returns `false` while `query.data` is undefined | ✅ Fixed |
| `ConfirmActionModalShell` extracted; `ActionConfirmModal` thin-wraps | ✅ Fixed |
| Protected `PaginationBar` still offers 100 in dropdown (clamped on select) | ✅ Accepted |
| `apiClient` shows error toast on non-success body for GET responses | ✅ Accepted |
| Protected table/pagination modules untouched | ✅ |

### Positive notes

- **Re-review:** Prior High findings addressed — server-table contract, loading availability, pageSize cap, shared confirm shell, `TableEmptyState`.
- **Feature wiring:** Availability enforcement is consistent at connect, distribute, upload, and video-detail surfaces.
- **UX:** Connected-but-disabled platforms remain visible with an "Unavailable" badge in Share Account.

---

## GitHub comments (Open findings)

No open findings — prior comments resolved on branch.

---

## Findings

---
Social platforms list bypasses frozen server-table controls

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: skillshow-admin-ui/src/pages/management/social-platforms/index.tsx  
Lines: 8-9, 38-44, 91-105

Description:
**Contract / DRY.** Initial diff hand-rolled pagination/sort and `socialPlatformTableSort.ts`.

**Re-verification (`ad3b8a59`):** ✅ **Fixed** — page uses `usePagination` with sort `filterKey`, caps `pageSize`, and delegates to `SocialPlatformsTable` with `setSort`. Table uses `applyServerSort`, hidden Ant pagination, and `PaginationBar`.

**PR comment (`index.tsx` line 38):** **Resolved** — `usePagination` + `PaginationBar` + `applyServerSort` wired per frozen contract.

---

---
Pagination bar allows pageSize 100 but API max is 50

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: skillshow-admin-ui/src/pages/management/social-platforms/index.tsx  
Lines: 40-44, 52, 97

Description:
**Contract.** Initial diff used `TeamTablePaginationBar` with global `PAGE_SIZE_OPTIONS` up to 100 while API max is 50.

**Re-verification (`ad3b8a59`):** ✅ **Fixed** — `cappedPageSize = Math.min(pageSize, SOCIAL_PLATFORM_LIST_MAX_PAGE_SIZE)` and `setPageSize` clamps before state update; API receives capped value.

**PR comment (`index.tsx` line 42):** **Resolved** — page size clamped to backend max before query and pagination state.

---

---
Disabled platforms treated as active while availability query is loading

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: skillshow-admin-ui/src/hooks/useSocialPlatformAvailability.ts  
Lines: 23-27

Description:
**Behavior.** Initial diff returned `true` when `!query.data`.

**Re-verification (`ad3b8a59`):** ✅ **Fixed:**

```23:27:skillshow-admin-ui/src/hooks/useSocialPlatformAvailability.ts
	const isVendorActive = useCallback(
		(vendorId: string): boolean => {
			if (!query.data) return false;
			return isVendorGloballyActive(vendorId, activeVendorIds);
```

**PR comment (`useSocialPlatformAvailability.ts` line 25):** **Resolved** — returns `false` until availability data loads.

---

---
ActionConfirmModal duplicates DestructiveActionConfirmModal structure

Risk Level: MEDIUM  
**Status:** ✅ Fixed  
File Path: skillshow-admin-ui/src/components/action-confirm-modal.tsx  
Lines: 1-13

Description:
**DRY.** Initial diff duplicated Modal/Drawer confirm layout.

**Re-verification (`ad3b8a59`):** ✅ **Fixed** — shared `confirm-action-modal-shell.tsx`; `ActionConfirmModal` is a thin wrapper.

**PR comment (`action-confirm-modal.tsx` line 12):** **Resolved** — confirm UI extracted to `ConfirmActionModalShell`.

---

---
Social platforms table missing TableEmptyState

Risk Level: MEDIUM  
**Status:** ✅ Fixed  
File Path: skillshow-admin-ui/src/pages/management/social-platforms/components/social-platforms-table.tsx  
Lines: 51

Description:
**Contract.** Initial diff omitted shared empty state.

**Re-verification (`ad3b8a59`):** ✅ **Fixed** — `locale={{ emptyText: emptyText ?? <TableEmptyState /> }}`.

**PR comment (`social-platforms-table.tsx` line 51):** **Resolved** — defaults to `TableEmptyState`.

---

---
PaginationBar dropdown still lists 100 while API max is 50

Risk Level: MEDIUM  
**Status:** Accepted  
File Path: skillshow-admin-ui/src/components/pagination-bar.tsx  
Lines: 32

Description:
**Contract (cosmetic).** `PaginationBar` (protected) still renders `PAGE_SIZE_OPTIONS` including 100. Social platforms page clamps `setPageSize` to 50, so selecting 100 stores 50 — behavior is correct but the dropdown offers a misleading option.

Impact:
- Minor UX confusion only; no incorrect API requests after clamp.

Recommendation:
Optional follow-up: add optional `pageSizeOptions` prop to `PaginationBar` in a dedicated protected-module ticket. Not a merge blocker for SKSH-330.

**PR comment:** N/A — Accepted (protected module; clamp mitigates behavior).

---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Social platforms list bypasses frozen server-table controls | HIGH | ✅ Fixed | skillshow-admin-ui/src/pages/management/social-platforms/index.tsx | 8-9, 38-44 | 38 |
| 2 | Pagination bar allows pageSize 100 but API max is 50 | HIGH | ✅ Fixed | skillshow-admin-ui/src/pages/management/social-platforms/index.tsx | 40-44 | 42 |
| 3 | Disabled platforms treated as active while availability query is loading | HIGH | ✅ Fixed | skillshow-admin-ui/src/hooks/useSocialPlatformAvailability.ts | 23-27 | 25 |
| 4 | ActionConfirmModal duplicates DestructiveActionConfirmModal structure | MEDIUM | ✅ Fixed | skillshow-admin-ui/src/components/action-confirm-modal.tsx | 1-13 | 12 |
| 5 | Social platforms table missing TableEmptyState | MEDIUM | ✅ Fixed | skillshow-admin-ui/src/pages/management/social-platforms/components/social-platforms-table.tsx | 51 | 51 |
| 6 | PaginationBar dropdown still lists 100 while API max is 50 | MEDIUM | Accepted | skillshow-admin-ui/src/components/pagination-bar.tsx | 32 | — |

**Merge readiness:** No open Critical/High/Medium blockers. Safe to merge from a frontend perspective.
