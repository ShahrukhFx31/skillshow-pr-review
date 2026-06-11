# Frontend PR Review — skillshow-admin-ui (`SKSH-314`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-314`  
**Base:** `main...HEAD` @ `5b2bd942`  
**Reviewed:** 2026-06-11  
**Re-reviewed:** 2026-06-11 (@ `5b2bd942` — `fix: improvements and fixes`)  
**Scope:** Server-driven **Linked Events** tab on app-user detail — `AppUserLinkedEventsTab`, `listAppUserLinkedEvents`, athlete-only tab gating; remove client-side `linkedEvents` from detail payload  
**Prompts:** `frontend-system-prompt.md`

**Files changed (vs `main`):** 10 — linked-events feature folder, `appUserService`, detail tabs/columns/types

**Findings:** 0 Open — **1 Fixed**

### Protected modules

No changes to frozen table-control, pagination, or audit-log modules. Wiring follows `useServerTableControls`, `applyServerSort`, `PaginationBar`, `TableEmptyState`.

### Positive notes

- Mock linked-events data removed; server-driven tab with debounced search and full `queryKey` deps.
- Athlete-only tab gating; sort keys align with backend allow-list.
- Desktop and mobile both use `eventRouteId` for event links (re-review fix).

---

## GitHub comments (Fixed)

### 1. `src/pages/management/app-users/onboarding/components/app-user-detail-columns.tsx` line 72 ✅

**Fixed (@ `5b2bd942`):** Event Name column now uses `buildEventViewPath(row.eventRouteId)` — matches mobile cards and slug-based event routes.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Desktop event link uses `eventId` instead of `eventRouteId` | MEDIUM | ✅ Fixed | src/pages/management/app-users/onboarding/components/app-user-detail-columns.tsx | 72 |

**Merge readiness:** **Approve for merge** — no open Critical/High/Medium findings.
