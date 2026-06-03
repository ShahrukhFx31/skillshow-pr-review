# SKSH-294 — Backend review (`skillshow`)

**Repo:** skillshow  
**Branch:** `SKSH-294`  
**Base:** `main...HEAD`  
**Initial review:** 2026-06-01 (`87785da`)  
**Re-reviewed:** 2026-06-03 (`6aac15c`) — fixes verified on branch  
**Scope:** Event lifecycle (`eventStatus`), listing status split (`listingStatus` / legacy `status`), `hostCode`, `streetAddress`, server-side list filters/search, removal of SS Event ID create/check API (Critical / High only)

**Review note:** Pagination / server-side list behavior (search coverage for `EV{n}`/slug, export scope, client sort vs server pages) is **out of scope** for this review—aligned with phased events list work.

---

## Overview

This PR reshapes the event domain: lifecycle status is stored as `eventStatus`, admin active/inactive is `listingStatus` (with legacy `status` fallback in filters and DTOs), `hostCode` is required and validated against active partners, and `streetAddress` is required on the schema. List endpoints gain `search`, `listingStatus`, `eventStatus`, and `eventType` query params with regex escaping in the repository. SS Event ID is deprecated (sparse optional field); the uniqueness check route is removed. Video-library event lookups are fixed to use `eventName` instead of `title`. Layering is generally sound (validation → controller → service → repository).

---

## Positive notes

- **Routes:** `listEventsQuerySchema` wired with `validate(..., "query")`; pagination capped via `PAGINATION.MAX_LIMIT`.
- **Repository:** `buildListFilter` uses `escapeRegexSource` for search; listing and lifecycle filters support legacy documents; combined filters use `$and` without clobbering `$or` (`58ca2fc`, `event.repository.test.ts`).
- **Service:** Create/update share `normalizeEventBody`; slug/seq duplicate retry on create (`c4ca523`); `hostCode` validated via `partnerRepository.existsActiveByHostCode`.
- **DTO:** `toEventDto` resolves listing/lifecycle defaults for legacy rows; list/detail/create/update return consistent `EventResponseDto`.
- **Tests:** `tests/repositories/event.repository.test.ts` covers legacy `eventStatus` / combined listing filters.

---

## Findings

---
Lifecycle `eventStatus` list filter ignores legacy documents

Risk Level: HIGH
File Path: src/repositories/event.repository.ts
Lines: 54-64

Description:
`listingStatus` filtering includes a fallback for documents that only have legacy `status`. `eventStatus` filtering uses a direct equality match with no fallback for documents missing `eventStatus` (pre-migration rows). `toEventDto` exposes `pending` via `resolveEventLifecycleStatus`, but those rows will not appear when `?eventStatus=pending` is used.

Impact:
- Incomplete result sets for filtered lists once the UI or integrators pass `eventStatus`.
- Inconsistent behavior between displayed lifecycle and filter results on legacy data.

Recommendation:
Mirror the listing pattern, e.g. when `query.eventStatus` is set:

```typescript
statusClauses.push({
  $or: [{ eventStatus: query.eventStatus }, legacyMissingEventStatus],
});
```

Or run a one-time migration to set `eventStatus` on all existing events before enabling the filter in production.

**PR comment (line 46):** **High:** `eventStatus` is a strict equality filter with no legacy fallback (unlike `listingStatus` above). Rows missing `eventStatus` won’t match `?eventStatus=pending` even though `toEventDto` defaults them to `pending`—please mirror the listing `$or` pattern or backfill the field.

**Re-review (2026-06-03):** **Fixed** in `58ca2fc` — `buildListFilter` adds `$or` with `{ eventStatus: { $exists: false } }` when filtering `pending`; non-default lifecycle uses impossible `_id` guard on legacy rows. Covered by `tests/repositories/event.repository.test.ts` (lines 63-111).
---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Lifecycle `eventStatus` list filter ignores legacy documents | HIGH | ✅ Fixed | src/repositories/event.repository.ts | 54-64 | 62 |

### Out of scope (pagination — not reported)

| Title | Status |
|--------|--------|
| List `search` does not match public `EV{n}` ID or `slug` | Accepted — deferred with server-paginated listing phase |

### Re-review notes (2026-06-03)

- Fix commit: `58ca2fc` (filter logic); branch tip `6aac15c` (includes main merge).
- **No new Critical or High** findings on the current backend diff.

**Merge readiness:** No open Critical/High blockers on the backend diff.
