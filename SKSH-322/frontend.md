# Frontend PR Review — skillshow-admin-ui (`SKSH-322`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-322`  
**Base:** `main...HEAD` @ `43a16d32`  
**Scope:** Public profile share page, account Settings tab (visibility toggle, email placeholder), share layout refactor, tab visibility hook (Critical / High / Medium)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 4 in scope (0 Critical, 0 High, 4 Medium) — **3 Open**, **1 Accepted**

### Protected modules

| Module | Status |
|--------|--------|
| `use-pagination.ts`, `pagination-bar.tsx`, `use-server-table-controls.ts`, `destructive-action-confirm-modal.tsx`, audit-log stack, `antd.adapter.tsx` | **Not modified** ✅ |

`ProfileVisibilityConfirmModal` correctly composes `DestructiveActionConfirmModal` ✅

### Files reviewed (30 changed)

Public profile share route/page/components, share video layout refactor (`PublicShareProfileAsideLayout`, `SharePageState`), profile settings tab + cards, account tab visibility hook, API routes/services/types, router entry.

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Reuse `PublicShareProfileCard` / display utils on profile + video share pages | ✅ Good |
| `mapLeanPublicShareVideoRowToDto` parity via shared API types | ✅ (backend owns mapping) |
| `SharePageState` extracted for loading/error shells | ✅ |
| `DestructiveActionConfirmModal` for visibility toggle | ✅ Contract |
| `useAccountTabVisibility` centralizes staff vs app-user tabs | ✅ |
| `isVideoPublic` used for profile visibility boolean | ⚠️ Semantic mismatch — see #2 |
| `usePagination(profileId, 0, …)` + custom Ant `Pagination` | ⚠️ Partial hook misuse — see #3 |
| SuperAdmin excluded from Connections/Sharing tabs | ⚠️ Behavior change — see #4 (Accepted if intentional) |
| Duplicate `PROFILE_PUBLIC_VIDEOS_PAGE_SIZE = 6` vs backend | ⚠️ Cross-stack literal — see #1 |

---

## GitHub comments (Open findings)

### 1. `src/pages/share/profile/constants.ts` line 2

**PR comment (line 2):** **Medium (Contract):** `PROFILE_PUBLIC_VIDEOS_PAGE_SIZE = 6` duplicates the backend constant. Add a short comment referencing `skillshow` `profile.constants.ts` (or a shared API contract doc) so a backend page-size change triggers a frontend review — the client does not send `pageSize`, but this value is still assumed in `usePagination`.

### 2. `src/pages/user/account/settings/components/PublicProfileSettingsCard.tsx` line 4

**PR comment (line 4):** **Medium (DRY / KISS):** `isVideoPublic` is video-domain naming for profile `isPublic`. Extract a neutral helper (e.g. `isExplicitlyPublic` in `@/utils` or account settings utils) or add `isProfilePublic` beside `isVideoPublic` to avoid misleading imports on profile settings UI.

### 3. `src/pages/share/profile/index.tsx` line 20

**PR comment (line 20):** **Medium (Contract / KISS):** `usePagination(profileId, 0, …)` hardcodes `total: 0`, so `bar`/`hidden` are unused and a stale `page` is not reset when `pagination.total` shrinks (empty grid on page 2+). Prefer `useState(1)` with reset on `profileId`, or pass `pagination.total` into `usePagination` and reset when `page > totalPages`.

---

## Findings

---
Duplicate server-fixed page size constant across repos

Risk Level: MEDIUM  
File Path: src/pages/share/profile/constants.ts  
Lines: 1-2

Description:
**Contract.** `PROFILE_PUBLIC_VIDEOS_PAGE_SIZE = 6` mirrors `skillshow` `profile.constants.ts`. The public API does not accept client `pageSize`, but the frontend still wires this into `usePagination` initial state. If backend changes page size, pagination UI and hook defaults can silently desync from `pagination.limit` returned by the API.

Impact:
- Cross-stack drift with no compile-time guard
- Reviewers may assume the frontend controls page size when it does not

Recommendation:
Keep the constant (documented as read-only contract) with an explicit cross-repo reference in the comment, or derive display-only defaults from `data.pagination.limit` after first fetch instead of a second magic number.

**PR comment (line 2):** **Medium (Contract):** Document that `6` must stay aligned with backend `PROFILE_PUBLIC_VIDEOS_PAGE_SIZE`; client must not send `pageSize`.

---

---
Profile settings card imports video-domain public helper

Risk Level: MEDIUM  
File Path: src/pages/user/account/settings/components/PublicProfileSettingsCard.tsx  
Lines: 4, 20, 34

Description:
**DRY / KISS.** `PublicProfileSettingsCard` imports `isVideoPublic` from `@/pages/videos/utils/video-public.utils` to coerce profile `isPublic`. Behavior is identical (`=== true`) but couples profile settings to video naming and misleads future readers.

Impact:
- Profile settings changes may incorrectly reuse video-specific utilities
- Harder to evolve profile vs video visibility rules independently

Recommendation:
Add a colocated helper, e.g. `isProfilePublic(isPublic?: boolean): boolean` in `settings/utils` or a shared `@/utils/boolean.utils.ts`, and use it in the settings card.

**PR comment (line 4):** **Medium (DRY):** Replace `isVideoPublic` with a profile-neutral `isProfilePublic` helper in settings code.

---

---
Public profile page misuses usePagination and can show stale empty pages

Risk Level: MEDIUM  
File Path: src/pages/share/profile/index.tsx  
Lines: 20, 63

Description:
**Contract / KISS.** `usePagination(profileId, 0, PROFILE_PUBLIC_VIDEOS_PAGE_SIZE)` passes `total: 0`, so only `page` / `setPage` are used while `bar` and `hidden` are dead. `PublicProfileVideoPagination` is a separate Ant `Pagination` implementation. If the user is on page 2+ and the video total drops below one page, the hook does not reset `page`, so the grid can show the empty state while pagination still allows navigating stale pages.

Impact:
- Confusing empty state when videos exist on page 1
- Unnecessary indirection vs `useState(1)` for a single query param

Recommendation:
Either:
- Use `const [page, setPage] = useState(1)` and `useEffect(() => setPage(1), [profileId])`, plus `useEffect` to clamp when `page > pagination.totalPages`, or
- Pass `pagination.total` into `usePagination` after data loads and call `resetPage()` when `page > totalPages`.

**PR comment (line 20):** **Medium:** `usePagination(..., 0, …)` bypasses hook pagination wiring — reset/clamp `page` when `pagination.total` changes.

---

---
SuperAdmin now excluded from Connections and Sharing Accounts tabs

**Status: Accepted** — refactor commit `a7e8f454` intentionally centralizes staff roles in `ACCOUNT_STAFF_ROLES` (includes `SuperAdmin`). Prior inline checks only excluded Admin/Crew/Editor. Confirm with product that super-admins should not see athlete/parent Connections or Sharing tabs; Settings tab remains available for public profile controls.

Risk Level: MEDIUM  
File Path: src/pages/user/account/constants/account-tab-visibility.constants.ts  
Lines: 4

Description:
**Global consistency / behavior change.** `ACCOUNT_STAFF_ROLES` adds `UserRole.SuperAdmin` to the set that hides Connections and Sharing Accounts. Previously, super-admin sessions could still see those tabs.

Impact:
- Super-admin UX change (not a runtime error)
- Aligns staff roles with admin/crew/editor for app-user-only tabs

Recommendation:
No code change required if product confirms. Otherwise remove `UserRole.SuperAdmin` from `ACCOUNT_STAFF_ROLES`.

**GitHub comment:** None required if behavior is intentional — document in ticket/release notes.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Duplicate server-fixed page size constant across repos | MEDIUM | Open | src/pages/share/profile/constants.ts | 1-2 |
| 2 | Profile settings card imports video-domain public helper | MEDIUM | Open | src/pages/user/account/settings/components/PublicProfileSettingsCard.tsx | 4 |
| 3 | Public profile page misuses usePagination and can show stale empty pages | MEDIUM | Open | src/pages/share/profile/index.tsx | 20 |
| 4 | SuperAdmin now excluded from Connections and Sharing Accounts tabs | MEDIUM | Accepted | src/pages/user/account/constants/account-tab-visibility.constants.ts | 4 |

**Merge readiness:** **Blocked on backend** — no frontend Critical/High blockers, but address backend High #2 (video filter mismatch) before merge to avoid broken profile grid links. Frontend Medium items are polish/clarity; fix #3 if QA hits stale pagination empty states.
