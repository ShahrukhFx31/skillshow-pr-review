# Backend PR Review — skillshow (`SKSH-314`)

**Repo:** skillshow  
**Branch:** `SKSH-314`  
**Base:** `main...HEAD` @ `a0e4e61`  
**Reviewed:** 2026-06-11  
**Re-reviewed:** 2026-06-11 (@ `a0e4e61` — `fix: improvements and fixes`)  
**Scope:** Server-driven app-user **Linked Events** list (`GET /v1/app-users/:appUserId/linked-events`); remove embedded `linkedEvents` from detail payload; source events from athlete registrations instead of video `eventId` distinct  
**Prompts:** `backend-system-prompt.md`

**Files changed (vs `main`):** 9 — `event-athlete.repository`, `app-user.service`, controller/routes/validation/types/constants, remove `video.repository.findDistinctEventIdsByUserLean`, controller test

**Findings:** 0 Open — **1 Fixed**

### Protected modules

No changes to frozen `list-query.validation.ts`, `list-query-aggregation.utils.ts`, change-stream, or audit-log modules.

### Positive notes

- `listLinkedEventsPaginatedByAthlete` now delegates to `runListQueryAggregate` with `buildRegexOrMatch` + `APP_USER_LINKED_EVENTS_LIST_SEARCH_FIELDS` (re-review fix).
- Controller uses `validatedQuery` + `DEFAULT_LIST_QUERY`; validation via `createListQuerySchema`.
- Non-athletes return empty paginated page; `linkedEvents` removed from detail payload.

---

## GitHub comments (Fixed)

### 1. `src/repositories/event-athlete.repository.ts` — ad-hoc `$facet` ✅

**Fixed (@ `a0e4e61`):** Refactored to `runListQueryAggregate` with lookup stages, `buildRegexOrMatch`, and `APP_USER_LINKED_EVENTS_LIST_SORT_FIELD_MAP`. Service passes full `AppUserLinkedEventsListQuery` to repository.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Ad-hoc `$facet` duplicates `runListQueryAggregate` | HIGH | ✅ Fixed | src/repositories/event-athlete.repository.ts | 134-161 |

**Merge readiness:** **Approve for merge** — no open Critical/High findings.
