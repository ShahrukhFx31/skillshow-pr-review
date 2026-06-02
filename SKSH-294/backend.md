# SKSH-294 — Backend review (`skillshow`)

**Repo:** skillshow  
**Branch:** `SKSH-294`  
**Base:** `main...HEAD`  
**Scope:** Event lifecycle (`eventStatus`), listing status split (`listingStatus` / legacy `status`), `hostCode`, `streetAddress`, server-side list filters/search, removal of SS Event ID create/check API (Critical / High only)

**Commit:** `87785da` — feat: enhance event management with lifecycle and listing statuses and added host field

**Review note:** Pagination / server-side list behavior (search coverage for `EV{n}`/slug, export scope, client sort vs server pages) is **out of scope** for this review—aligned with phased events list work.

---

## Overview

This PR reshapes the event domain: lifecycle status is stored as `eventStatus`, admin active/inactive is `listingStatus` (with legacy `status` fallback in filters and DTOs), `hostCode` is required and validated against active partners, and `streetAddress` is required on the schema. List endpoints gain `search`, `listingStatus`, `eventStatus`, and `eventType` query params with regex escaping in the repository. SS Event ID is deprecated (sparse optional field); the uniqueness check route is removed. Video-library event lookups are fixed to use `eventName` instead of `title`. Layering is generally sound (validation → controller → service → repository).

---

## Positive notes

- **Routes:** `listEventsQuerySchema` wired with `validate(..., "query")`; pagination capped via `PAGINATION.MAX_LIMIT`.
- **Repository:** `buildListFilter` uses `escapeRegexSource` for search; listing filter supports legacy documents via `$or` on `listingStatus` / `status`.
- **Service:** Create/update share `normalizeEventBody` (date/state normalization, `status` → `listingStatus`, `location` build); `hostCode` validated via `partnerRepository.existsActiveByHostCode`.
- **DTO:** `toEventDto` resolves listing/lifecycle defaults for legacy rows; list/detail/create/update return consistent `EventResponseDto`.
- **Tests:** Controller, validation, and model tests updated for new fields; hostCode rejection covered.

---

## Findings

---
Lifecycle `eventStatus` list filter ignores legacy documents

Risk Level: HIGH
File Path: src/repositories/event.repository.ts
Lines: 45-47

Description:
`listingStatus` filtering includes a fallback for documents that only have legacy `status`. `eventStatus` filtering uses a direct equality match with no fallback for documents missing `eventStatus` (pre-migration rows). `toEventDto` exposes `pending` via `resolveEventLifecycleStatus`, but those rows will not appear when `?eventStatus=pending` is used.

Impact:
- Incomplete result sets for filtered lists once the UI or integrators pass `eventStatus`.
- Inconsistent behavior between displayed lifecycle and filter results on legacy data.

Recommendation:
Mirror the listing pattern, e.g. when `query.eventStatus` is set:

```typescript
filter["$or"] = [
  { eventStatus: query.eventStatus },
  {
    eventStatus: { $exists: false },
    ...(query.eventStatus === EVENT_DEFAULT_LIFECYCLE_STATUS
      ? {} /* match legacy rows with no field */
      : { _id: { $exists: false } }), // or omit filter if no safe default
  },
];
```

Or run a one-time migration to set `eventStatus` on all existing events before enabling the filter in production.

**PR comment (line 46):** **High:** `eventStatus` is a strict equality filter with no legacy fallback (unlike `listingStatus` above). Rows missing `eventStatus` won’t match `?eventStatus=pending` even though `toEventDto` defaults them to `pending`—please mirror the listing `$or` pattern or backfill the field.
---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Lifecycle `eventStatus` list filter ignores legacy documents | HIGH | Open | src/repositories/event.repository.ts | 45-47 | 46 |

### Out of scope (pagination — not reported)

| Title | Status |
|--------|--------|
| List `search` does not match public `EV{n}` ID or `slug` | Accepted — deferred with server-paginated listing phase |
