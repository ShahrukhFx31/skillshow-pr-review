# Frontend PR Review — skillshow-admin-ui (`SKSH-189`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-189`  
**Base:** `main...HEAD`  
**Re-verified:** `65186b4e` (`fix: improvement caching logic`)  
**Scope:** React performance, hooks, JSX/props, Tailwind/file structure (Critical, High, Medium only)  
**Findings:** 4 (0 Critical, 2 High fixed, 1 Medium fixed, 1 Medium accepted) — **0 open**

---

## Verification summary

| # | Title | Risk | Status | Evidence |
|---|--------|------|--------|----------|
| 1 | Stale cache after patch → view | HIGH | **Fixed** | `crew-user-form.tsx` 137-150: `setQueryData` on list + detail, then invalidate |
| 2 | Redundant full list fetch on view/edit | HIGH | **Fixed** | `onboarding/index.tsx`: only `getCrewUser`; `selectedCrewUser` from detail |
| 3 | Bulk update leaves detail cache stale | MEDIUM | **Fixed** | `crew-users-bulk-update.tsx` 22-24: predicate invalidates all `crew-users` queries |
| 4 | Failed detail fetch renders empty form | MEDIUM | **Accepted** | Team decision; not blocking SKSH-189 |

**Regression re-check (PR diff):** **0 new Critical or High** issues in the caching fix commit.

---

---
Patch success navigates to view with stale React Query cache

**Status: Fixed**

Risk Level: HIGH  
File Path: src/pages/management/crew-users/onboarding/components/crew-user-form.tsx  
Lines: 137-150

**Verification:**  
`patchCrewUserMutation.onSuccess` now calls `setQueryData` for `CREW_USERS_LIST_QUERY_KEY` and `["management", "crew-users", "detail", userId]`, preserves table `key` via `mergeRow`, then invalidates both keys before navigating to view.

---

---
View/edit onboarding loads full crew list plus detail

**Status: Fixed**

Risk Level: HIGH  
File Path: src/pages/management/crew-users/onboarding/index.tsx  
Lines: 31-40

**Verification:**  
`listCrewUsers` query removed. View/edit uses only `getCrewUser` with detail query key; `selectedCrewUser` is `fetchedCrewUser` only.

---

---
Bulk/detail query keys not updated after bulk patch

**Status: Fixed**

Risk Level: MEDIUM  
File Path: src/pages/management/crew-users/dashboard/components/crew-users-bulk-update.tsx  
Lines: 21-25

**Verification:**  
`onSuccess` invalidates with predicate `queryKey[0] === "management" && queryKey[1] === "crew-users"`, covering list, detail, and reporting-manager keys (per original recommendation).

---

---
Failed detail fetch still renders onboarding form shell

**Status: Accepted (for now)** — team decision; not blocking SKSH-189.

Risk Level: MEDIUM  
File Path: src/pages/management/crew-users/onboarding/index.tsx  
Lines: 50, 125-133

Description:
`isInitialDataReady` is `selectedCrewUser !== undefined || !isDetailPending`. After removing the list fallback, a failed or 404 `getCrewUser` leaves `selectedCrewUser` undefined while `isDetailPending` is false, so the page mounts `CrewUserForm` with empty `initialValues` instead of an error or not-found state.

Impact:
- Deep links or stale IDs show a blank edit/view form rather than a clear error
- Worse than the prior list+fallback pattern, which could still resolve the row from dashboard cache

Recommendation:
Gate readiness on success and surface errors:

```tsx
const { data: fetchedCrewUser, isPending: isDetailPending, isError } = useQuery({ ... });

const isInitialDataReady =
  mode === CrewUserFormMode.Add || selectedCrewUser !== undefined;

// In JSX: isError && !isDetailPending → DataNotFound or Result status="404"
```

**PR comment (line 50):**  
**Medium:** When `getCrewUser` fails, `isInitialDataReady` is still true and we render an empty form. Please handle `isError` / missing row (not-found UI) now that we no longer fall back to the full list cache.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Stale cache after patch → view | HIGH | Fixed | `onboarding/components/crew-user-form.tsx` | 137-150 |
| 2 | Redundant full list fetch on view/edit | HIGH | Fixed | `onboarding/index.tsx` | 31-40 |
| 3 | Bulk update leaves detail cache stale | MEDIUM | Fixed | `dashboard/components/crew-users-bulk-update.tsx` | 21-25 |
| 4 | Failed detail fetch renders empty form | MEDIUM | Accepted | `onboarding/index.tsx` | 50, 125-133 |

**Positive notes:** Caching fix commit is focused (3 files). `mergeRow` preserves Ant table `key` when patching list cache. Feature layout and styling conventions unchanged. No new JSX `? : null` patterns in the fix.

**Skipped (per severity / rules):** Bulk invalidation also refetches reporting managers (minor extra request, acceptable). Delete flow still invalidates list only (matches team users). Ad-hoc Tailwind / Ant `!` overrides remain out of scope.
