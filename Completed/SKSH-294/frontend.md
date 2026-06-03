# SKSH-294 — Frontend review (`skillshow-admin-ui`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-294`  
**Base:** `main...HEAD`  
**Initial review:** 2026-06-01 (`c1685de5`)  
**Re-reviewed:** 2026-06-03 (`0f40c258`) — fixes verified on branch  
**Scope:** Event lifecycle status UI, host field, street address, server-driven events list, removal of SS Event ID flow (Critical / High / Medium)

**Review note:** Pagination-related items (client column sort on a single server page, full-dataset CSV export vs `pagedRows`) are **out of scope** for this review—aligned with phased events dashboard list work.

---

## Overview

The events dashboard moves from loading all events client-side to paginated `fetchEventsList` with debounced server `search` and `listingStatus` / `eventType` filters. Forms replace SS Event ID with `eventStatus`, `hostCode` (partner-driven select), and `streetAddress`. Edit/view loads a single event via `getEventBySlug` instead of the full list. Table status column shows stored lifecycle status instead of date-derived “Live / Upcoming / Completed”.

---

## Positive notes

- **Detail load:** Edit/view uses `getEventBySlug` — avoids downloading the full catalog per form.
- **Host field:** `listPartnersDirectory` with `EVENT_HOST_OPTIONS_QUERY_KEY` and 5m `staleTime` (`77ea4b34`).
- **Lifecycle display:** `unknown` / “Unset” in constants, tags, CSV, and sort order; `coerceEventLifecycleStatus` no longer maps missing values to `pending`.
- **Types/utils:** `sanitizeEventRecord` and list query builder updated for `eventStatus`, `listingStatus`, `hostCode`, `streetAddress`.
- **Create payload:** `isAssignableEventLifecycleStatus` prevents sending `unknown` on create.

---

## Findings

---
Legacy API rows display lifecycle status as Pending

Risk Level: MEDIUM
File Path: src/pages/events/utils/event.utils.ts
Lines: 59-74

Description:
`coerceEventLifecycleStatus` defaults unknown/missing `eventStatus` to `pending`. Legacy events without `eventStatus` in MongoDB will show “Pending” in the table and detail views even if their real operational state differs.

Impact:
- Incorrect lifecycle labels until data is backfilled.
- Risk of wrong operational decisions from the admin table.

Recommendation:
Use a distinct fallback (e.g. `"unknown"` with label “Unset”) until backfill, or drive display from a migration-aware API field. Coordinate with backend backfill of `eventStatus`.

**PR comment (line 49):** **Medium:** Missing `eventStatus` defaults to `pending`, so legacy rows may show the wrong lifecycle until backfill. Consider an explicit unset value/label or coordinate with a DB migration.

**Re-review (2026-06-03):** **Fixed** in `77ea4b34` — missing/null/invalid `eventStatus` maps to `EVENT_LIFECYCLE_STATUS.unknown` with label “Unset” in table/tags/CSV; create uses `isAssignableEventLifecycleStatus` so `unknown` is not posted. **Note:** API responses from `toEventDto` still emit `pending` for DB rows without `eventStatus`; the UI shows “Unset” when the field is absent/invalid in the raw payload. Aligning backend DTO with `unknown` is optional follow-up, not a blocker.
---
---
`listPartners()` loads full partner list for host select

Risk Level: MEDIUM
File Path: src/pages/events/onboarding/components/event-details-section.tsx
Lines: 24-28

Description:
`EventDetailsSection` calls `listPartners()` (unpaginated `GET` partners) on every form that includes the section. Host options are derived in memory from the full array.

Impact:
- Slower form mount and higher memory use as the partner directory grows.
- Extra load on the partners API for bulk-update drawer and create/edit forms.

Recommendation:
Use a dedicated lightweight endpoint (e.g. active host codes only) or `listPartnersDirectory` if it is smaller; cache with a longer `staleTime` and share query key across forms.

**PR comment (line 23):** **Medium:** Host select loads the full `listPartners()` list on every form mount—consider a host-codes-only endpoint, `listPartnersDirectory`, or longer `staleTime` as the directory grows.

**Re-review (2026-06-03):** **Fixed** — uses `listPartnersDirectory`, `EVENT_HOST_OPTIONS_QUERY_KEY`, and `EVENT_HOST_OPTIONS_STALE_TIME_MS` (5 minutes).
---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Legacy API rows display lifecycle status as Pending | MEDIUM | ✅ Fixed | src/pages/events/utils/event.utils.ts | 59-74 | 64 |
| 2 | `listPartners()` loads full partner list for host select | MEDIUM | ✅ Fixed | src/pages/events/onboarding/components/event-details-section.tsx | 24-28 | 25 |

### Out of scope (pagination — not reported)

| Title | Status |
|--------|--------|
| Table column sort only re-orders the current server page | Accepted — deferred with server-paginated listing |
| CSV export exports only the current page | Accepted — deferred with server-paginated listing |

### Re-review notes (2026-06-03)

- Fix commits: `77ea4b34` (unknown lifecycle + host options), `8dad86c8` (form layout); branch tip `0f40c258`.
- **No new Critical, High, or Medium** findings on the current frontend diff.

**Merge readiness:** No open findings; pagination items remain Accepted out of scope.
