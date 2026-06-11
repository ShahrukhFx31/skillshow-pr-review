# Frontend PR Review — skillshow-admin-ui (`SKSH-260`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-260`  
**Base:** `main...HEAD` @ `5a6d4e35`  
**Re-reviewed:** 2026-06-11  
**Scope:** Wire crew **Performance Reviews** to live API; crew dashboard UX (reporting manager, rating column, `/dashboard` route); remove mock feedback data  
**Prompts:** `frontend-system-prompt.md`

**Files changed (vs `main`):** 25 — performance-reviews hook/service/types, `CrewPerformanceReviews`, crew dashboard table/columns/KPI, router + auth redirects, realtime invalidation

**Findings:** 0 Open — **3 Accepted**, **1 Fixed**

### Protected modules

No changes to frozen `use-server-table-controls`, `use-pagination`, `pagination-bar`, `table-sort`, or audit-log modules.

### Positive notes

- Mock feedback data removed; account tab loads live API with skeleton loading.
- `useCrewPerformanceReviews` includes `userId` and `page` in `queryKey`; realtime invalidation covers `crew-performance-reviews`.
- Role-based `/dashboard` entry unifies crew/editor with other roles.
- Comment truncation + `PerformanceReviewCommentModal` for long feedback text.

---

## GitHub comments (Accepted — intentional)

### 1. `src/router/index.tsx` line 319

**Accepted (2026-06-11):** No legacy `/dashboard/crew` redirect — crew uses `/dashboard` via role-based `Dashboard` router only.

### 2. `src/pages/user/account/general/crew/hooks/use-crew-performance-reviews.ts` line 16

**Accepted (2026-06-11):** Performance reviews use `DEFAULT_LIST_PAGINATION.pageSize` (5) from global config — intentional; not a dedicated 10-row feature constant.

---

## GitHub comments (Fixed)

### 3. `src/pages/user/account/general/crew/hooks/use-crew-performance-reviews.ts` line 16

**Fixed:** Duplicate local `CREW_PERFORMANCE_REVIEWS_PAGE_SIZE` removed; hook and `CrewPerformanceReviews.tsx` line 144 both use `DEFAULT_LIST_PAGINATION` from `@/constants/common.constant`.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Legacy `/dashboard/crew` route removed without redirect | MEDIUM | Accepted | src/router/index.tsx | 319 |
| 2 | Performance reviews uses global `DEFAULT_LIST_PAGINATION.pageSize` (5) | MEDIUM | Accepted | src/pages/user/account/general/crew/hooks/use-crew-performance-reviews.ts | 16 |
| 3 | Duplicate page-size constant | MEDIUM | ✅ Fixed | src/pages/user/account/general/crew/hooks/use-crew-performance-reviews.ts | 16 |

**Merge readiness:** **Approve for merge** — no open Critical/High/Medium findings.
