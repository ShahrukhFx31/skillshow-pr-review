# Frontend PR Review — skillshow-admin-ui (`SKSH-311`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-311`  
**Base:** `main...HEAD`  
**Re-verified:** 2026-06-05 (full `/pr-review`) — @ `e8b3e6e2`  
**Scope reviewed:** **50 files** in `git diff main...HEAD`. **SKSH-311 core (21 files):** app-user activity server table, coach teams dashboard, `appUserService`, edit-request status constants/mapping. **Also in branch diff:** destructive-modal consolidation, permission/role/policy delete modals, connections/share-account/video-library/my-videos action hooks (DRY modal migration — reviewed for regressions).  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency enforced)

**Aligned with:** [backend.md](./backend.md)

### Full review coverage

| File group | Reviewed |
|------------|----------|
| `app-users/activity/*` (index, types, utils, actions, columns, table, responsive) | ✅ |
| `app-users/onboarding/index.tsx` | ✅ |
| `app-users/teams/dashboard/*` | ✅ |
| `api/services/appUserService.ts`, `api/services/editRequestService.ts` | ✅ |
| `editRequest/constants`, `EditRequestListToolbar` (`EDIT_REQUEST_STATUSES` export) | ✅ |
| Destructive-modal migration (connections, share-account, ConnectSocialModal, video-library, management action hooks) | ✅ Spot-checked |

### pr-review counts

| Metric | Count |
|--------|-------|
| New findings this pass | 1 |
| Prior Open → Fixed this pass | 3 |
| Prior Open → Accepted this pass | 1 |
| Remaining Open | 0 |

---

## GitHub comments (Open findings)

None.

---

---
Edit-request status filter sends UI values to API

Risk Level: HIGH  
File Path: src/pages/management/app-users/activity/index.tsx  
Lines: 42-57

Description:
**DRY / Global consistency.** The edit-requests tab passed `status: statusFilter` directly to `listAppUserActivityEditRequests` without mapping UI keys to backend values.

Impact:
- Selecting UI-only values (e.g. `pending_review`) triggered **400** validation errors.

Recommendation:
Map through `mapUiStatusFilterToBackendStatus` before the API call; omit param when mapper returns `undefined`.

**Re-review (`e8b3e6e2`):** ✅ **Fixed** — `backendEditRequestStatusFilter` useMemo (lines 42-45) and conditional spread at lines 55-57.

**PR comment:** Resolved on branch.

**Status:** ✅ Fixed

---

---
Edit-request filter options include non-API status keys

Risk Level: HIGH  
File Path: src/pages/management/app-users/activity/utils.ts  
Lines: 18-22

Description:
**DRY / Global consistency.** `getEditRequestStatusFilterOptions` iterated all `EDIT_REQUEST_STATUS_LABELS` keys, including UI-only statuses the server list API does not accept.

Recommendation:
Use shared `EDIT_REQUEST_STATUSES` from `editRequest.constants.ts`.

**Re-review (`e8b3e6e2`):** ✅ **Fixed** — `EDIT_REQUEST_STATUSES` exported to constants (shared with `EditRequestListToolbar`); activity utils maps curated list at lines 18-22.

**PR comment:** Resolved on branch.

**Status:** ✅ Fixed

---

---
Coach teams meta query fetches full page payload for total only

Risk Level: MEDIUM  
File Path: src/pages/management/app-users/teams/dashboard/index.tsx  
Lines: 54

Description:
`allTeamsMeta` used full `DEFAULT_LIST_QUERY` (10 rows) only to read `pagination.total` for `hasNoTeams`.

Recommendation:
Use `pageSize: 1` for the meta fetch.

**Re-review (`e8b3e6e2`):** ✅ **Fixed** — line 54: `{ ...DEFAULT_LIST_QUERY, pageSize: 1 }`.

**PR comment:** Resolved on branch.

**Status:** ✅ Fixed

---

---
UI-only `failed` status filter fetches unfiltered list

Risk Level: MEDIUM  
File Path: src/pages/management/app-users/activity/index.tsx  
Lines: 42-57

Description:
**KISS / UX.** `EDIT_REQUEST_STATUSES` includes `failed`, but `mapUiStatusFilterToBackendStatus` returns `undefined` for `failed` (no backend equivalent). When selected, `backendEditRequestStatusFilter` is `undefined` and `listAppUserActivityEditRequests` runs **without** a status filter — returning all edit requests while the UI implies a failed-only view.

Impact:
- Misleading filter UX: “Request Failed” shows the full list.
- Same pattern may affect other server-filtered edit-request surfaces using `EDIT_REQUEST_STATUSES` + `failed`.

Recommendation:
For server-filtered activity tab, exclude `failed` from filter options (or disable with tooltip), or keep `failed` only on client-filtered athlete list pages.

**PR comment (anchor `src/pages/management/app-users/activity/index.tsx` lines 42-57):** Selecting **Request Failed** returns `undefined` from `mapUiStatusFilterToBackendStatus` (lines 42-45), so the API call at lines 55-57 omits `status` and returns all rows. Remove or disable `failed` in server-filtered activity filter options.

**Accepted (2026-06-05):** Intentional — `failed` is UI-only with no backend status; keeping it in the shared `EDIT_REQUEST_STATUSES` list for parity with the athlete edit-request list. Omitting the query param (unfiltered fetch) is acceptable for this admin activity slice; server-side `failed` filtering deferred to a follow-up if a backend status is introduced.

**Status:** Accepted

---

## Positive notes

- **Prior blockers fixed** (`e8b3e6e2`): UI→backend status mapping, curated `EDIT_REQUEST_STATUSES`, coach teams meta `pageSize: 1`.
- **`mapUiStatusFilterToBackendStatus` improved:** `accepted` now maps to `"accepted"` (not `submitted`) — pairs better with backend once filter semantics are fixed.
- **Server table migration:** Activity uses `useServerTableControls`, `applyServerSort`, `PaginationBar`; coach teams match same pattern.
- **Modal consolidation:** `DestructiveActionConfirmModal` replaces bespoke delete/unlink modals across connections, share-account, ConnectSocialModal, video-library, management action hooks — DRY win; spot-check shows explicit `title`/`description` on unlink flows.
- **Videos tab:** `getVideoStatusFilterOptions` keys match backend `videoStatus` validation.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Edit-request status filter sends UI values to API | HIGH | ✅ Fixed | `src/pages/management/app-users/activity/index.tsx` | 42-57 |
| 2 | Edit-request filter options include non-API status keys | HIGH | ✅ Fixed | `src/pages/management/app-users/activity/utils.ts` | 18-22 |
| 3 | Coach teams meta query fetches full page payload for total only | MEDIUM | ✅ Fixed | `src/pages/management/app-users/teams/dashboard/index.tsx` | 54 |
| 4 | UI-only `failed` status filter fetches unfiltered list | MEDIUM | Accepted | `src/pages/management/app-users/activity/index.tsx` | 42-57 |

**Merge readiness:** ✅ No open Critical/High/Medium blockers. Finding #4 accepted per team — shared `EDIT_REQUEST_STATUSES` includes UI-only `failed`; unfiltered fetch when selected is acceptable for SKSH-311.
