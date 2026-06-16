# Backend PR Review — skillshow (`SKSH-349`)

**Repo:** skillshow — `https://github.com/fx31labs-mvp/skillshow.git`  
**Branch:** `SKSH-349`  
**Base:** `main...HEAD` @ `8545cb3`  
**Initial review:** 2026-06-16  
**Re-review:** 2026-06-16  
**Scope:** Event create/update/import validation — reject start/end dates older than 25 years; city must contain at least one letter (Critical / High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules / Security)

**Aligned with:** [frontend.md](./frontend.md)

**Findings:** 0 (0 Critical, 0 High) — **0 Open**

### Protected modules

| Module | Status |
|--------|--------|
| `list-query.validation.ts`, `list-query-aggregation.utils.ts`, audit-log stack, change-stream modules | **Not modified** ✅ |

### Security scan (`SECURITY-AUDIT-PRE-RELEASE.md`)

| Check | Verdict |
|-------|---------|
| No route, auth, middleware, upload, or export changes | ✅ |
| Validation hardening only (dates, city) on existing guarded event routes | ✅ |
| Import path uses `validateCreateEvent` → same Joi schema as HTTP create | ✅ |
| No weakened `authorize`, IDOR, or S3 paths touched | ✅ |

### Files reviewed

| File | Change |
|------|--------|
| `src/constants/event.constants.ts` | `EVENT_IMPORT_MAX_PAST_YEARS = 25` |
| `src/utils/event-import.utils.ts` | `eventImportEarliestAllowedDate`, `isEventDateWithinImportRange`, message helper |
| `src/validation/event.validation.ts` | `eventCityFieldSchema`, `eventDateFieldSchema`; wired into `createEventSchema` / `updateEventSchema` |
| `tests/utils/event-import.utils.test.ts` | Past-year window unit tests |
| `tests/validation/event.validation.test.ts` | City letter + date range Joi tests |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| `eventDateFieldSchema` / `eventCityFieldSchema` dedupe inline Joi on create; `updateEventSchema` inherits via `fork` | ✅ DRY |
| Range helpers live in `event-import.utils.ts`; constant in `event.constants.ts` | ✅ |
| Import validator calls `validateCreateEvent` — no parallel validation path | ✅ |
| UTC date parsing via `parseEventDatePayload` + UTC `dayjs` in range helper | ✅ |
| Frontend mirrors `EVENT_MAX_PAST_YEARS = 25` and matching message shape | ✅ (see frontend re-review) |
| Service `normalizeEventBody` does not re-check range/city (Joi on route + import validator runs first) | ✅ Accepted (existing pattern) |

### Positive notes

- Helpers are testable in isolation; boundary case (`2001-06-16` from ref `2026-06-16`) covered in unit test.
- City rule is intentionally stricter than `eventName` (letter-only vs letter-or-digit).
- Existing events with legacy dates remain editable when dates are not changed (partial `updateEventSchema` patches).

---

## GitHub comments

_No Critical or High findings._

---

## Findings

_No Critical or High findings._

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|

**Merge readiness:** ✅ No Critical or High blockers on the backend. Validation is centralized in `createEventSchema` and flows through HTTP routes and import validation.
