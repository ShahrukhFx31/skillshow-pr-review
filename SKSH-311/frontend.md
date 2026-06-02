# Frontend PR Review — skillshow-admin-ui (`SKSH-311`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-311`  
**Base:** `main...HEAD`  
**Scope:** Server-side tables for partners, crew users, and SkillShow team users (Critical, High, Medium)  
**Findings:** 4 (1 Critical, 2 High, 1 Medium)

**Aligned with:** [backend.md](./backend.md) (directory API contract)

---

---
`listPartnersDirectory` incompatible with paginated directory API

Risk Level: CRITICAL  
File Path: src/api/services/partnerService.ts  
Lines: 21-25

Description:
`listPartnersDirectory()` still types and consumes `GET .../partners/directory` as `PartnerRow[]`. The backend branch returns `PaginatedResponse<PartnerListRow>` with default `pageSize: 10`.

`ConnectSocialModal` sets `const { data: allPartners = [] } = useQuery({ queryFn: listPartnersDirectory })` and calls `allPartners.filter(...)`.

Impact:
- Quick Connect partner tabs break at runtime (`.filter` is not a function on `{ items, pagination }`) or show no partners.
- Users cannot browse/connect to partners beyond the first page even after unwrapping `items`.

Recommendation:
Align with the backend directory contract: either unwrap `items` and paginate/fetch all pages, or call a dedicated unpaginated directory endpoint. Example if the API stays paginated for directory temporarily:

```ts
export async function listPartnersDirectory(): Promise<PartnerRow[]> {
  const { items } = await apiClient.get<PartnerListResponse>({
    params: { page: 1, pageSize: 100, status: "active" },
    url: `${PARTNERS_API_PREFIX}/directory`,
  });
  return items;
}
```

Prefer a backend flat directory response for this use case (see backend finding #1).

**PR comment (line 22):** **Critical:** Directory now returns `{ items, pagination }` but this helper still expects `PartnerRow[]`. Connect Social will break—unwrap `items` or use an unpaginated directory API.

---

---
Partner edit/view loads partner from first list page only

Risk Level: HIGH  
File Path: src/pages/partners/onboarding/index.tsx  
Lines: 33-43

Description:
Edit and view modes resolve `selectedPartner` via `listPartners({ ...DEFAULT_LIST_QUERY })`, which only requests `page: 1`, `pageSize: 10`. There is no `getPartner(partnerId)` fallback (unlike team users, which use `getSkillshowUser`).

Impact:
- Opening `/partners/edit/:partnerId` or `/partners/view/:partnerId` for a partner not on page 1 shows a perpetual loading/empty state or wrong “not found” behavior after pagination.

Recommendation:
Add `GET /v1/partners/:partnerId` on the backend (if missing) and a `getPartner(partnerId)` client, or fetch by id through an existing detail endpoint. At minimum, pass `partnerId` as a search/filter param if the API supports it, or request with `pageSize: MAX` only as a stopgap—not for production scale.

**PR comment (line 35):** **High:** Edit/view still resolve the partner from `listPartners` with default page size 10. Partners off page 1 won’t load—use a by-id fetch like team user onboarding does with `getSkillshowUser`.

---

---
Reporting manager options built from paginated list (10 rows)

Risk Level: HIGH  
File Path: src/pages/management/skillshow-users/onboarding/components/team-user-form.tsx  
Lines: 103-125

Description:
`reportingManagerOptions` is derived from `listSkillshowUsers({ ...DEFAULT_LIST_QUERY })` → `data.items` (10 users). `buildReportingManagerOptions` only sees that subset.

Impact:
- Add/edit team user forms omit valid reporting managers who are not on the first page of the default sort.
- Admins may assign incorrect managers or think options are complete.

Recommendation:
Use a dedicated reporting-managers endpoint (crew users already have `/reporting-managers`), extend it for team users, or query with a purpose-built API (e.g. search/select endpoint) instead of the dashboard list page.

**PR comment (line 104):** **High:** Reporting manager dropdown is built from the paginated list (10 rows). Most managers won’t appear—use a dedicated options endpoint or an unpaginated manager query.

---

---
Role filter options derived from current page only

Risk Level: MEDIUM  
File Path: src/pages/management/crew-users/dashboard/index.tsx  
Lines: 55, 59

Description:
`crewUsers` is assigned from `data?.items` (line 55), so `buildRoleFilterOptions(crewUsers)` (line 59) only sees the current server page. Dynamic role values not present on the active page are missing from the filter dropdown. The same pattern applies on team users (`teamUsers = data?.items`, lines 67–72) and partners (`partners = data?.items`, line 55 in `partners/dashboard/index.tsx`).

Impact:
- Filters cannot target roles/types that exist in the dataset but not on the current page.
- Admins may believe a role does not exist when it is simply not in the current slice.

Recommendation:
Source filter enums from constants/backend metadata, or add a small “filter facets” API. Until then, fetch facet values with a separate query (e.g. `pageSize: 100` + distinct) that is not tied to the table’s current page.

**PR comment (line 55):** **Medium:** `crewUsers` is now `data.items` (one page) but `buildRoleFilterOptions(crewUsers)` still treats it like a full list—roles on other pages won’t appear in the filter. Same on team users (67–72) and partners (`partners/dashboard/index.tsx` line 51); consider constants or a facets endpoint.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | `listPartnersDirectory` vs paginated directory API | CRITICAL | Open | src/api/services/partnerService.ts | 21-25 |
| 2 | Partner edit/view uses paginated list only | HIGH | Open | src/pages/partners/onboarding/index.tsx | 33-43 |
| 3 | Reporting managers from paginated list | HIGH | Open | src/pages/management/skillshow-users/onboarding/components/team-user-form.tsx | 103-125 |
| 4 | Role/type filter options from current page | MEDIUM | Open | src/pages/management/crew-users/dashboard/index.tsx | 55, 59 |

## Positive notes

- `useServerTableControls` centralizes debounced search, sort, and page reset on filter changes.
- Column `key` / `dataIndex` align with backend `sortBy` (`fullName`, `createdAt`, `crewDesignation`).
- Tables use server sort via `applyServerSort` and external `PaginationBar`; client-side filtering removed appropriately.
- Shared `PaginatedResponse` / `ListQuery` types mirror the API.

**Merge readiness:** Blocked on Critical #1 (and matching backend directory fix). Resolve High #2–#3 before merge; Medium #4 can follow if product accepts limited filter labels short-term.
