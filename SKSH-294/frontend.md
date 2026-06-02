# SKSH-294 — Frontend review (`skillshow-admin-ui`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-294`  
**Base:** `main...HEAD`  
**Scope:** Event lifecycle status UI, host field, street address, server-driven events list, removal of SS Event ID flow (Critical / High / Medium)

**Commit:** `c1685de5` — refactor: introduce event lifecycle status, added host field and removed the event ID

**Review note:** Pagination-related items (client column sort on a single server page, full-dataset CSV export vs `pagedRows`) are **out of scope** for this review—aligned with phased events dashboard list work.

---

## Overview

The events dashboard moves from loading all events client-side to paginated `fetchEventsList` with debounced server `search` and `listingStatus` / `eventType` filters. Forms replace SS Event ID with `eventStatus`, `hostCode` (partner-driven select), and `streetAddress`. Edit/view loads a single event via `getEventBySlug` instead of the full list. Table status column shows stored lifecycle status instead of date-derived “Live / Upcoming / Completed”.

---

## Positive notes

- **Detail load:** Edit/view uses `getEventBySlug` — avoids downloading the full catalog per form.
- **Host field:** Options built from active partners with `EVENT_HOST_CODE_PATTERN`; aligns with backend Joi.
- **Types/utils:** `sanitizeEventRecord` and list query builder updated for `eventStatus`, `listingStatus`, `hostCode`, `streetAddress`.
- **Empty state:** `probeEventsExist` avoids fetching all rows just to detect zero events.
- **Loading:** `SkySectionLoading` for form/detail panels keeps page chrome visible.

---

## Findings

---
Legacy API rows display lifecycle status as Pending

Risk Level: MEDIUM
File Path: src/pages/events/utils/event.utils.ts
Lines: 45-49

Description:
`coerceEventLifecycleStatus` defaults unknown/missing `eventStatus` to `pending`. Legacy events without `eventStatus` in MongoDB will show “Pending” in the table and detail views even if their real operational state differs.

Impact:
- Incorrect lifecycle labels until data is backfilled.
- Risk of wrong operational decisions from the admin table.

Recommendation:
Use a distinct fallback (e.g. `"unknown"` with label “Unset”) until backfill, or drive display from a migration-aware API field. Coordinate with backend backfill of `eventStatus`.

**PR comment (line 49):** **Medium:** Missing `eventStatus` defaults to `pending`, so legacy rows may show the wrong lifecycle until backfill. Consider an explicit unset value/label or coordinate with a DB migration.
---
---
`listPartners()` loads full partner list for host select

Risk Level: MEDIUM
File Path: src/pages/events/onboarding/components/event-details-section.tsx
Lines: 21-24

Description:
`EventDetailsSection` calls `listPartners()` (unpaginated `GET` partners) on every form that includes the section. Host options are derived in memory from the full array.

Impact:
- Slower form mount and higher memory use as the partner directory grows.
- Extra load on the partners API for bulk-update drawer and create/edit forms.

Recommendation:
Use a dedicated lightweight endpoint (e.g. active host codes only) or `listPartnersDirectory` if it is smaller; cache with a longer `staleTime` and share query key across forms.

**PR comment (line 23):** **Medium:** Host select loads the full `listPartners()` list on every form mount—consider a host-codes-only endpoint, `listPartnersDirectory`, or longer `staleTime` as the directory grows.
---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Legacy API rows display lifecycle status as Pending | MEDIUM | Open | src/pages/events/utils/event.utils.ts | 45-49 | 49 |
| 2 | `listPartners()` loads full partner list for host select | MEDIUM | Open | src/pages/events/onboarding/components/event-details-section.tsx | 21-24 | 23 |

### Out of scope (pagination — not reported)

| Title | Status |
|--------|--------|
| Table column sort only re-orders the current server page | Accepted — deferred with server-paginated listing |
| CSV export exports only the current page | Accepted — deferred with server-paginated listing |
