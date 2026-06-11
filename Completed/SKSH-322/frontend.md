# Frontend PR Review — skillshow-admin-ui (`SKSH-322`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-322`  
**Base:** `main...HEAD` @ `9a8df8c2`  
**Initial review:** 2026-06-11 @ `43a16d32`  
**Re-review:** 2026-06-11 @ `9a8df8c2` (`feat: improve public profile sharing and settings logic`)  
**Scope:** Public profile share page, account Settings tab (visibility toggle, email placeholder), share layout refactor, tab visibility hook (Critical / High / Medium)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 4 (0 Critical, 0 High, 4 Medium) — **0 Open**, **3 Fixed**, **1 Accepted**

### Protected modules

| Module | Status |
|--------|--------|
| `use-pagination.ts`, `pagination-bar.tsx`, `use-server-table-controls.ts`, `destructive-action-confirm-modal.tsx`, audit-log stack, `antd.adapter.tsx` | **Not modified** ✅ |

`ProfileVisibilityConfirmModal` correctly composes `DestructiveActionConfirmModal` ✅

### Files reviewed (31 changed)

Public profile share route/page/components, share video layout refactor (`PublicShareProfileAsideLayout`, `SharePageState`), profile settings tab + cards + `profile-public.utils`, account tab visibility hook, API routes/services/types, router entry.

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Reuse `PublicShareProfileCard` / display utils on profile + video share pages | ✅ Good |
| `SharePageState` extracted for loading/error shells | ✅ |
| `DestructiveActionConfirmModal` for visibility toggle | ✅ Contract |
| `useAccountTabVisibility` centralizes staff vs app-user tabs | ✅ |
| `isProfilePublic` in settings utils (not `isVideoPublic`) | ✅ Fixed |
| Profile share page pagination state | ✅ Fixed — `useState` + clamp |
| `PROFILE_PUBLIC_VIDEOS_PAGE_SIZE` in constants | ✅ Accepted — doc-only, no hook coupling |
| SuperAdmin excluded from Connections/Sharing tabs | ✅ Accepted — intentional |

### Positive notes

- Re-review replaces `usePagination(profileId, 0, …)` with `useState` + profileId reset + `totalPages` clamp.
- `isProfilePublic` colocated under `settings/utils/profile-public.utils.ts`.

---

## GitHub comments

No open findings — prior comments resolved on branch.

---

## Findings

---
Duplicate server-fixed page size constant across repos

Risk Level: MEDIUM  
**Status:** Accepted  
File Path: src/pages/share/profile/constants.ts  
Lines: 1-2

Description:
**Contract.** `PROFILE_PUBLIC_VIDEOS_PAGE_SIZE = 6` mirrors backend `profile.constants.ts`. Re-review removed hook coupling — constant is now documentation-only (not imported by `index.tsx`). Comment already states server-fixed page size and that client must not send `pageSize`.

Impact:
- Low drift risk; constant serves as API contract reference

Recommendation:
Optional cleanup: remove unused export or add `skillshow/src/constants/profile.constants.ts` path in comment. Not required to merge.

---

---
Profile settings card imports video-domain public helper

Risk Level: MEDIUM  
**Status:** ✅ Fixed @ `9a8df8c2`  
File Path: src/pages/user/account/settings/components/PublicProfileSettingsCard.tsx  
Lines: 5, 20, 34

Description:
**DRY / KISS.** Previously imported `isVideoPublic` for profile `isPublic`.

Recommendation:
Use profile-neutral helper — **done** via `settings/utils/profile-public.utils.ts` (`isProfilePublic`), also wired in `settings-tab.tsx`.

---

---
Public profile page misuses usePagination and can show stale empty pages

Risk Level: MEDIUM  
**Status:** ✅ Fixed @ `9a8df8c2`  
File Path: src/pages/share/profile/index.tsx  
Lines: 20-40

Description:
**Contract / KISS.** Previously used `usePagination(profileId, 0, …)` with dead `bar`/`hidden` and no page clamp.

Recommendation:
Use `useState(1)` with profileId reset and clamp when `page > totalPages` — **done**:

```typescript
const [page, setPage] = useState(DEFAULT_LIST_PAGINATION.page);
useEffect(() => setPage(DEFAULT_LIST_PAGINATION.page), [profileId]);
useEffect(() => {
  const totalPages = data?.pagination?.totalPages;
  if (totalPages == null) return;
  const maxPage = Math.max(DEFAULT_LIST_PAGINATION.page, totalPages);
  if (page > maxPage) setPage(maxPage);
}, [data?.pagination?.totalPages, page]);
```

---

---
SuperAdmin now excluded from Connections and Sharing Accounts tabs

Risk Level: MEDIUM  
**Status:** Accepted  
File Path: src/pages/user/account/constants/account-tab-visibility.constants.ts  
Lines: 4

Description:
**Global consistency / behavior change.** `ACCOUNT_STAFF_ROLES` includes `SuperAdmin`, hiding Connections and Sharing Accounts for staff. Settings tab remains for public profile controls. Intentional per refactor commit `a7e8f454`.

**GitHub comment:** None — accepted behavior.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Duplicate server-fixed page size constant across repos | MEDIUM | Accepted | src/pages/share/profile/constants.ts | 1-2 |
| 2 | Profile settings card imports video-domain public helper | MEDIUM | ✅ Fixed | src/pages/user/account/settings/components/PublicProfileSettingsCard.tsx | 5 |
| 3 | Public profile page misuses usePagination and can show stale empty pages | MEDIUM | ✅ Fixed | src/pages/share/profile/index.tsx | 20-40 |
| 4 | SuperAdmin now excluded from Connections and Sharing Accounts tabs | MEDIUM | Accepted | src/pages/user/account/constants/account-tab-visibility.constants.ts | 4 |

**Merge readiness:** **Approve for merge** — no open Critical/High/Medium blockers.
