# Frontend PR Review — skillshow-admin-ui (`SKSH-311`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-311`  
**Base:** `main...HEAD`  
**Re-verified:** 2026-06-02 (`/pr-review` full pass) — @ `origin/SKSH-311` → `cbab2339`  
**Scope reviewed:** All **21 files** in `git diff main...HEAD` — app-user activity server table (videos + edit requests), coach teams dashboard server table + filter options, `appUserService` API updates, `in_progress` edit-request status labels.  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency enforced)

**Aligned with:** [backend.md](./backend.md)

### Full review coverage

| File group | Reviewed |
|------------|----------|
| `app-users/activity/*` (index, types, utils, actions, columns, table, responsive) | ✅ |
| `app-users/onboarding/index.tsx` (activity panel wiring) | ✅ |
| `app-users/teams/dashboard/*` (index, types, utils, actions, columns, table, hooks) | ✅ |
| `api/services/appUserService.ts` | ✅ |
| `editRequest/constants`, `editRequest/types`, `adminEditRequest/constants` (`in_progress`) | ✅ |

### pr-review counts

| Metric | Count |
|--------|-------|
| New findings this pass | 0 |
| Prior Open → Fixed this pass | 0 |
| Remaining Open | 3 (2 High, 1 Medium) |

---

## GitHub comments (Open findings)

### Finding 1 — `src/pages/management/app-users/activity/index.tsx` line 50

> Edit-request status filter sends UI keys like `pending_review` to the new paginated API (`status: statusFilter` at line 50), but the backend only accepts DB status values (`submitted`, etc.). Map through `mapUiStatusFilterToBackendStatus` before calling `listAppUserActivityEditRequests`, same as the existing edit-request service contract.

The edit-requests tab sends the raw UI status (`pending_review`, `failed`, `changes_requested_for_editor`, etc.) as the `status` query param. The backend Joi schema only accepts `EDIT_REQUEST_STATUS_VALUES` (e.g. `submitted`, not `pending_review`), so several filter selections will return **400** validation errors. Reuse the existing `mapUiStatusFilterToBackendStatus` helper from `editRequestService.ts` (same pattern documented for UI-only statuses) before calling `listAppUserActivityEditRequests`, and omit the param when the mapper returns `undefined` (e.g. `failed`).

### Finding 2 — `src/pages/management/app-users/activity/utils.ts` line 19

> `getEditRequestStatusFilterOptions` iterates all `EDIT_REQUEST_STATUS_LABELS` keys at line 19 (`Object.entries(EDIT_REQUEST_STATUS_LABELS)`), including UI-only values the API rejects. Use the same curated status list as `EditRequestListToolbar` so the dropdown only shows server-filterable options.

`getEditRequestStatusFilterOptions` builds options from the full `EDIT_REQUEST_STATUS_LABELS` map, which includes UI-only keys not valid for the new server list API. Mirror `EditRequestListToolbar`'s curated `EDIT_REQUEST_STATUSES` array instead of `Object.entries(EDIT_REQUEST_STATUS_LABELS)`, so the dropdown only offers filterable statuses and stays aligned with `mapUiStatusFilterToBackendStatus`.

### Finding 3 — `src/pages/management/app-users/teams/dashboard/index.tsx` line 54

> The `allTeamsMeta` query uses full `DEFAULT_LIST_QUERY` at line 54 (`{ ...DEFAULT_LIST_QUERY }`, 10 rows) just to check `pagination.total`. Use `pageSize: 1` for the meta fetch since only the total is needed for `hasNoTeams`.

A second `listCoachTeams` query with `DEFAULT_LIST_QUERY` runs on every coach teams page load solely to read `pagination.total` for the empty-state gate (`hasNoTeams`). That fetches up to `pageSize` team rows (default 10) when only the count is needed.

---

---
Edit-request status filter sends UI values to API

Risk Level: HIGH  
File Path: src/pages/management/app-users/activity/index.tsx  
Lines: 48-51

Description:
**DRY / Global consistency.** The edit-requests tab passes `status: statusFilter` directly to `listAppUserActivityEditRequests`. Filter values come from `getEditRequestStatusFilterOptions()`, which exposes UI status keys (`pending_review`, `failed`, `changes_requested_for_editor`, …). The backend list endpoint validates `status` against `EDIT_REQUEST_STATUS_VALUES` (DB/API names like `submitted`, `editor_assigned`). The codebase already has `mapUiStatusFilterToBackendStatus` in `editRequestService.ts` for this mapping, but this page does not use it.

Impact:
- Selecting **Pending Admin Review** (`pending_review`) or other UI-only values triggers a **400** from the API and breaks the activity table.
- Users cannot filter edit requests server-side despite the new endpoint.

Recommendation:
Map UI → backend before the API call and skip `undefined` mappings:

```typescript
import { mapUiStatusFilterToBackendStatus } from "@/api/services/editRequestService";
import type { EditRequestStatus } from "@/pages/editRequest/types/editRequest.types";

// inside queryFn for edit-requests branch:
const backendStatus = statusFilter
  ? mapUiStatusFilterToBackendStatus(statusFilter as EditRequestStatus)
  : undefined;

return listAppUserActivityEditRequests(userId, {
  ...listParams,
  ...(backendStatus ? { status: backendStatus } : {}),
});
```

**PR comment (anchor `src/pages/management/app-users/activity/index.tsx` line 50):** Edit-request status filter sends UI keys like `pending_review` to the new paginated API (`status: statusFilter` at line 50), but the backend only accepts DB status values (`submitted`, etc.). Map through `mapUiStatusFilterToBackendStatus` before calling `listAppUserActivityEditRequests`, same as the existing edit-request service contract.

**Status:** Open

---

---
Edit-request filter options include non-API status keys

Risk Level: HIGH  
File Path: src/pages/management/app-users/activity/utils.ts  
Lines: 18-19

Description:
**DRY / Global consistency.** `getEditRequestStatusFilterOptions` returns every key in `EDIT_REQUEST_STATUS_LABELS`, including UI-only statuses (`pending_review`, `failed`, `changes_requested_for_editor`) that the server list endpoint does not accept. `EditRequestListToolbar` already defines a curated `EDIT_REQUEST_STATUSES` list for filter UX; this activity page diverges from that pattern after migrating to server-side tabulation.

Impact:
- Dropdown offers statuses that cannot be queried server-side (400 or no-op).
- Inconsistent filter behavior vs other edit-request surfaces.

Recommendation:
Export or colocate a shared filterable-status list (e.g. reuse `EDIT_REQUEST_STATUSES` from `EditRequestListToolbar` or move it to `editRequest.constants.ts`) and build options from that:

```typescript
export function getEditRequestStatusFilterOptions(): { label: string; value: string }[] {
  return EDIT_REQUEST_STATUSES.map((status) => ({
    value: status,
    label: EDIT_REQUEST_STATUS_LABELS[status],
  }));
}
```

Pair with `mapUiStatusFilterToBackendStatus` in `activity/index.tsx`.

**PR comment (anchor `src/pages/management/app-users/activity/utils.ts` line 19):** `getEditRequestStatusFilterOptions` iterates all `EDIT_REQUEST_STATUS_LABELS` keys at line 19 (`Object.entries(EDIT_REQUEST_STATUS_LABELS)`), including UI-only values the API rejects. Use the same curated status list as `EditRequestListToolbar` so the dropdown only shows server-filterable options.

**Status:** Open

---

---
Coach teams meta query fetches full page payload for total only

Risk Level: MEDIUM  
File Path: src/pages/management/app-users/teams/dashboard/index.tsx  
Lines: 54

Description:
A second `listCoachTeams` query with `DEFAULT_LIST_QUERY` runs on every coach teams page load solely to read `pagination.total` for the empty-state gate (`hasNoTeams`). That fetches up to `pageSize` team rows (default 10) when only the count is needed.

Impact:
- Extra network and DB work on each visit; scales poorly for coaches with many teams.

Recommendation:
Request `pageSize: 1` (or add a lightweight count endpoint) for the meta query:

```typescript
queryFn: () => listCoachTeams(userId as string, { ...DEFAULT_LIST_QUERY, pageSize: 1 }),
```

**PR comment (anchor `src/pages/management/app-users/teams/dashboard/index.tsx` line 54):** The `allTeamsMeta` query uses full `DEFAULT_LIST_QUERY` at line 54 (`{ ...DEFAULT_LIST_QUERY }`, 10 rows) just to check `pagination.total`. Use `pageSize: 1` for the meta fetch since only the total is needed for `hasNoTeams`.

**Status:** Open

---

## Positive notes

- **Server table migration:** Activity panel correctly uses `useServerTableControls`, `applyServerSort`, hidden Ant pagination + `PaginationBar`, and tab-scoped `queryKey` — aligned with events/My Videos dashboards.
- **API wiring:** `appUserService` adds `listAppUserActivityVideos`, `listAppUserActivityEditRequests`, `listCoachTeams`, and `getCoachTeamFilterOptions`; onboarding passes `userId` into `AppUserActivityPanel`.
- **Coach teams:** Server-side search/sort/filters, filter-options endpoint, and non-coach guard match backend contracts; `teamId` sort key maps to backend `seq`.
- **Videos tab:** `getVideoStatusFilterOptions` uses `VIDEO_LIST_STATUS_FILTERS` — keys match backend `videoStatus` validation.
- **Display mapping:** `getEditRequestActivityStatusLabel` uses `mapBackendStatusToUiStatus` for row tags — display layer is correct; only the filter/query path needs the reverse mapping.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Edit-request status filter sends UI values to API | HIGH | Open | `src/pages/management/app-users/activity/index.tsx` | 48-51 |
| 2 | Edit-request filter options include non-API status keys | HIGH | Open | `src/pages/management/app-users/activity/utils.ts` | 18-19 |
| 3 | Coach teams meta query fetches full page payload for total only | MEDIUM | Open | `src/pages/management/app-users/teams/dashboard/index.tsx` | 54 |

**Merge readiness:** ❌ Blocked — 2 open **High** frontend findings on edit-request status filtering, plus **3 High backend findings** on raw DB status / filter semantics / phantom `in_progress` (see [backend.md](./backend.md)). Fix both sides together; Medium meta-query optimization is optional.
