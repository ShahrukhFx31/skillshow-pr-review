# Frontend PR Review — skillshow-admin-ui (`SKSH-243`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-243`  
**Base:** `main...HEAD` (re-reviewed at `264c0bbc`)  
**Scope:** React performance, hooks, memory, architecture, JSX/props, Tailwind/file structure (Critical, High, Medium only)  
**Findings:** 5 (0 Critical, 1 High, 3 Medium fixed, 1 Accepted)

---

---
Table `Spin` indicator logo size regressed from `sm` to `md`

Risk Level: HIGH
File Path: src/components/loading/table-loading.tsx
Lines: 6-10

Description:
`TableSkySpinIndicator` previously rendered `<SkyLogoMark size="sm" />` (32×40px). It now uses `SKY_LOGO_DEFAULT_SIZE`, which is `"md"` (48×60px). That constant is the generic default for `SkyLogoMark`, not the table-overlay size.

Impact:
- Every consumer of `tableSkySpinProps` / `tableSkyLoading` (management tables, permission tables, `VideoTabsTable`, etc.) shows a larger in-table spinner than before
- Unintentional visual change outside the files touched in this PR

Recommendation:
Introduce a dedicated constant (e.g. `SKY_TABLE_SPIN_LOGO_SIZE: SkyLogoMarkSize = "sm"`) and use it only in `table-loading.tsx`. Keep `SKY_LOGO_DEFAULT_SIZE` for standalone `SkyLogoMark` usage.

**Re-review:** ✅ Fixed — `SKY_TABLE_SPIN_LOGO_SIZE = "sm"`; unchanged through `264c0bbc`.

---

---
Empty video list treats fetch errors as “no uploads”

Risk Level: MEDIUM
File Path: src/pages/editRequest/components/SelectFromMyVideosModal.tsx
Lines: 56-74, 231-244

Description:
The modal only destructures `{ data, isLoading }` from `useQuery`. On failure, `isLoading` is false, `data` is undefined, `rows` is empty, and `hasActiveVideoFilters` is false, so `emptyVideosMessage` becomes `EDIT_REQUEST_SELECT_MY_VIDEOS_EMPTY_NO_UPLOADS` (“No videos uploaded yet.”).

Impact:
- Users see a misleading empty state when the API fails (network/server), not when they have zero uploads
- Harder to diagnose real upload vs. load problems

Recommendation:
Handle `isError` (and optionally `refetch`) from `useQuery`: show an error/retry UI instead of the no-uploads copy when the request failed.

**Re-review:** ✅ Fixed — `isError` / `isFetching` / `refetch`; error panel + retry; list/empty gated with `!isError`.

---

---
Page loader height tied to magic `7.5rem` offset

Risk Level: MEDIUM
File Path: src/components/loading/sky-loading.css
Lines: 112-116

Description:
`.ss-loader-page` uses `min-height: calc(100dvh - 7.5rem)` to sit below header/nav. That offset is not derived from layout tokens or shared header variables.

Impact:
- Loader vertical centering drifts if header/nav height or dashboard shell padding changes
- Subtle layout bugs on responsive or theme updates without a code change in the loader CSS

Recommendation:
Prefer a CSS variable set on the dashboard layout (e.g. `--dashboard-chrome-height`) and reference it in `.ss-loader-page`, or use flex `flex-1` / `min-h-0` on `<Main>` so the loader fills the content region without a hard-coded subtraction.

**Re-review:** ✅ Fixed — `calc(100dvh - var(--dashboard-chrome-height))`; `:root` fallback in `global.css`; horizontal layout override on dashboard `Layout`.

---

---
Unrelated account form change bundled in loading PR

Risk Level: MEDIUM
File Path: src/pages/user/account/general/components/BasicInformationEditForm.tsx
Lines: 66-71

Description:
This PR removes the chevron `Icon` from metric suffix inputs and changes suffix styling (`pointer-events-none`, `bg-gray-100!`, etc.). That is unrelated to loading variants/constants.

Impact:
- Mixed concerns in one PR complicate review and rollback
- Account profile UX changes ship with loader work without explicit QA on that surface

Recommendation:
Revert or split into a separate ticket/PR focused on account form UX.

**Status:** Accepted — intentional in this PR (suffix is non-interactive; chevron removed to avoid looking like a dropdown).

---

---
Edit-request list initial load reverted to table `Spin` instead of `SkyPageLoading`

Risk Level: MEDIUM
File Path: src/pages/editRequest/index.tsx
Lines: 224-225

Description:
After merging `main` (`73dfc818`), the edit-request list route again returned a centered Ant `Spin` with `tableSkySpinProps(true)` for `showInitialLoading` instead of `SkyPageLoading`.

Impact:
- Inconsistent full-page loading UX on edit-request vs. other dashboard pages
- Table-overlay spin used for a full-page bootstrap state
- Page loader a11y not applied on this route

Recommendation:
Restore `return <SkyPageLoading />` when `showInitialLoading` is true; keep `isPending && !data` guards.

**Re-review:** ✅ Fixed in `264c0bbc` — `SkyPageLoading` restored; unused `Spin` / `tableSkySpinProps` imports removed. `showInitialLoading` guard unchanged.

---

## Re-review summary (`264c0bbc`)

All five summary rows are **Fixed** or **Accepted**. No new **Critical**, **High**, or **Medium** issues in the fix commit.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Table `Spin` logo size `sm` → `md` regression | HIGH | ✅ Fixed | `table-loading.tsx` / `loading.constants.ts` | 6-10, 21-22 |
| 2 | Fetch error shown as “no uploads” | MEDIUM | ✅ Fixed | `SelectFromMyVideosModal.tsx` | 56-74, 231-244 |
| 3 | Magic `7.5rem` page loader offset | MEDIUM | ✅ Fixed | `sky-loading.css` / `global.css` / `dashboard/index.tsx` | 112-116 |
| 4 | Unrelated `BasicInformationEditForm` diff | MEDIUM | Accepted | `BasicInformationEditForm.tsx` | 66-71 |
| 5 | Edit-request list uses table `Spin` not `SkyPageLoading` | MEDIUM | ✅ Fixed | `editRequest/index.tsx` | 224-225 |

**Positive notes:** Loading variants (`fullscreen` / `page` / `section`), constants/types, table spin size split, dashboard chrome height token, modal + video list error/retry UX, and consistent `SkyPageLoading` on edit-request list, video details, and dashboard suspense.

**Skipped (per request):** Pagination-related issues.

**Merge readiness:** No open Critical/High/Medium blockers.
