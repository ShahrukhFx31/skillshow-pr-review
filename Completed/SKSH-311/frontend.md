# Frontend PR Review — skillshow-admin-ui (`SKSH-311`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-311`  
**Base:** `main...HEAD`  
**Re-verified:** 2026-06-08 (full `/pr-review`) — @ `32a9b4e3`  
**Archived:** 2026-06-08  
**Scope reviewed:** **50 files** in `git diff main...HEAD`. **SKSH-311 core (21 files):** app-user activity server table, coach teams dashboard, `appUserService`, edit-request status constants/mapping. **Also in branch diff:** destructive-modal consolidation, permission/role/policy delete modals, connections/share-account/video-library/my-videos/management action hooks (DRY modal migration).  
**Prompts:** `frontend-system-prompt.md`, `backend-system-prompt.md` (DRY / KISS / Global consistency enforced)

**Aligned with:** [backend.md](./backend.md)

### Full review coverage

| File group | Reviewed |
|------------|----------|
| `app-users/activity/*` (index, types, utils, actions, columns, table, responsive) | ✅ |
| `app-users/onboarding/index.tsx` | ✅ |
| `app-users/teams/dashboard/*` | ✅ |
| `api/services/appUserService.ts`, `api/services/editRequestService.ts` | ✅ |
| `editRequest/constants`, `EditRequestListToolbar` (`EDIT_REQUEST_STATUSES` export) | ✅ |
| Destructive-modal migration (~29 files) | ✅ Spot-checked — delete flows use `itemName` or explicit `title`/`description` |

### pr-review counts

| Metric | Count |
|--------|-------|
| New findings this pass | 0 |
| Prior Open → Fixed (re-verified) | 3 |
| Prior Open → Accepted (re-verified) | 1 |
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
**DRY / Global consistency.** Edit-requests tab passed raw UI `statusFilter` to the paginated API.

Impact:
- UI-only values (e.g. `pending_review`) triggered **400** validation errors.

Recommendation:
Map through `mapUiStatusFilterToBackendStatus` before the API call.

**Re-review (`e8b3e6e2`, confirmed @ `32a9b4e3`):** ✅ **Fixed** — `backendEditRequestStatusFilter` useMemo (lines 42-45) and conditional spread at lines 55-57.

**PR comment:** Resolved on branch.

**Status:** ✅ Fixed

---

---
Edit-request filter options include non-API status keys

Risk Level: HIGH  
File Path: src/pages/management/app-users/activity/utils.ts  
Lines: 18-22

Description:
**DRY / Global consistency.** Filter options built from full `EDIT_REQUEST_STATUS_LABELS` including UI-only keys.

Recommendation:
Use shared `EDIT_REQUEST_STATUSES` from `editRequest.constants.ts`.

**Re-review (`e8b3e6e2`, confirmed @ `32a9b4e3`):** ✅ **Fixed** — `EDIT_REQUEST_STATUSES` exported to constants; activity utils maps curated list at lines 18-22; `EditRequestListToolbar` imports same list.

**PR comment:** Resolved on branch.

**Status:** ✅ Fixed

---

---
Coach teams meta query fetches full page payload for total only

Risk Level: MEDIUM  
File Path: src/pages/management/app-users/teams/dashboard/index.tsx  
Lines: 54

Description:
`allTeamsMeta` used full `DEFAULT_LIST_QUERY` only to read `pagination.total`.

Recommendation:
Use `pageSize: 1` for the meta fetch.

**Re-review (`e8b3e6e2`, confirmed @ `32a9b4e3`):** ✅ **Fixed** — line 54: `{ ...DEFAULT_LIST_QUERY, pageSize: 1 }`.

**PR comment:** Resolved on branch.

**Status:** ✅ Fixed

---

---
UI-only `failed` status filter fetches unfiltered list

Risk Level: MEDIUM  
File Path: src/pages/management/app-users/activity/index.tsx  
Lines: 42-57

Description:
**KISS / UX.** `failed` has no backend equivalent; mapper returns `undefined`, so API runs unfiltered.

Impact:
- “Request Failed” filter shows all rows.

Recommendation:
Exclude `failed` from server-filtered activity options, or disable with tooltip.

**PR comment (anchor `src/pages/management/app-users/activity/index.tsx` lines 42-57):** Selecting **Request Failed** returns `undefined` from `mapUiStatusFilterToBackendStatus` (lines 42-45), so the API call at lines 55-57 omits `status` and returns all rows.

**Accepted (2026-06-05):** Intentional — `failed` kept in shared `EDIT_REQUEST_STATUSES` for parity with athlete edit-request list; unfiltered fetch acceptable for this admin slice.

**Status:** Accepted

---

## Positive notes

- **Server table migration:** Activity + coach teams use `useServerTableControls`, `applyServerSort`, hidden Ant pagination + `PaginationBar`.
- **Status mapping:** `mapUiStatusFilterToBackendStatus` maps `pending_review` → `submitted`, `accepted` → `accepted`; omits UI-only `failed`.
- **Modal consolidation:** `DestructiveActionConfirmModal` replaces bespoke delete/unlink modals — DRY win; management delete flows pass `itemName`; unlink flows pass explicit `title`/`description`.
- **Videos tab:** `getVideoStatusFilterOptions` keys match backend `videoStatus` validation.
- **Display:** `getEditRequestActivityStatusLabel` uses `mapBackendStatusToUiStatus` for row tags.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Edit-request status filter sends UI values to API | HIGH | ✅ Fixed | `src/pages/management/app-users/activity/index.tsx` | 42-57 |
| 2 | Edit-request filter options include non-API status keys | HIGH | ✅ Fixed | `src/pages/management/app-users/activity/utils.ts` | 18-22 |
| 3 | Coach teams meta query fetches full page payload for total only | MEDIUM | ✅ Fixed | `src/pages/management/app-users/teams/dashboard/index.tsx` | 54 |
| 4 | UI-only `failed` status filter fetches unfiltered list | MEDIUM | Accepted | `src/pages/management/app-users/activity/index.tsx` | 42-57 |

**Merge readiness:** ✅ No open Critical/High/Medium blockers. All findings Fixed or Accepted; no new issues in `main...HEAD` at `32a9b4e3`.
