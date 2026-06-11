# Frontend PR Review — skillshow-admin-ui (`SKSH-314`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-314`  
**Base:** `main...HEAD` @ `7f861e85`  
**Reviewed:** 2026-06-11  
**Scope:** Server-driven **Linked Events** tab on app-user detail — `AppUserLinkedEventsTab`, `listAppUserLinkedEvents`, athlete-only tab gating; remove client-side `linkedEvents` from detail payload  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Files changed (vs `main`):** 10 — linked-events feature folder, `appUserService`, detail tabs/columns/types

**Findings:** 1 Open (0 Critical, 0 High, 1 Medium)

### Protected modules

No changes to frozen `use-server-table-controls`, `use-pagination`, `pagination-bar`, `table-sort`, or audit-log modules. Consumer wiring is correct.

### Positive notes

- Uses `useServerTableControls` with `filterKey` reset, debounced search, and full `queryKey` deps.
- Desktop table uses `applyServerSort`, hidden pagination + `PaginationBar`, and `TableEmptyState` — matches coach-teams pattern.
- Athlete-only tab filtering (`APP_USER_ROLE.athlete`) and `resolvedTab` guard prevent non-athletes from landing on linked-events.
- `linkedEvents` removed from `AppUserDetail` type; data fetched lazily per tab.
- Sort keys (`createdAt`, `date`, `eventName`) align with backend `APP_USER_LINKED_EVENTS_LIST_SORT_BY_VALUES`.

---

## GitHub comments (Open findings)

### 1. `src/pages/management/app-users/linked-events/components/app-user-linked-events-table-responsive.tsx` line 22

**PR-diff anchor** (line 22 is in the diff; `app-user-detail-columns.tsx:72` is unchanged context — GitHub won't accept an inline there). Mobile uses `row.eventRouteId` here, but the shared desktop column in `app-user-detail-columns.tsx` still links with `row.eventId` (unchanged line ~72). Update the column render to `buildEventViewPath(row.eventRouteId)` so desktop and mobile match.

---

---
Desktop event link uses `eventId` instead of `eventRouteId`

Risk Level: MEDIUM  
File Path: src/pages/management/app-users/linked-events/components/app-user-linked-events-table-responsive.tsx (PR comment anchor); fix in `app-user-detail-columns.tsx` ~72 (unchanged — not commentable inline)  
Lines: 22 (anchor); ~71-74 (fix target)

Description:
**Global consistency.** The PR adds `eventRouteId` to the API row shape and uses it in `AppUserLinkedEventsTableResponsive`, but `APP_USER_LINKED_EVENT_COLUMNS` still links with `row.eventId`. Mongo ids resolve via `getEventByRef`, but slug-based routes are the established pattern elsewhere (event view sections, mobile cards in this PR).

Impact:
- Desktop and mobile linked-event links can diverge (ObjectId path vs slug path) for the same row.
- Inconsistent URLs when events have slugs; harder to match event view deep links.

Recommendation:
Update the column render to use `eventRouteId`:

```tsx
render: (name: string, row) => (
  <Link className="text-inherit!" to={buildEventViewPath(row.eventRouteId)}>
    {name}
  </Link>
),
```

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Desktop event link uses `eventId` instead of `eventRouteId` | MEDIUM | Open | app-user-linked-events-table-responsive.tsx (anchor) / app-user-detail-columns.tsx (fix) | 22 / ~72 |

**Merge readiness:** **Approve with minor fix** — no Critical/High blockers; 1 Medium global-consistency fix recommended before merge.
