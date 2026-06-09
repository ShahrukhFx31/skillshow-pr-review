# Frontend PR Review — skillshow-admin-ui (`SKSH-326`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-326`  
**Base:** `main...HEAD` @ `c9cdfe19`  
**Scope:** Unify upload/edit event dropdown on `GET /v1/events/me` for athletes and coaches; remove coach-only video-library lookups path (Critical, High, Medium only)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 3 (0 Critical, 0 High, 3 Medium) — **3 Open**

### Protected modules

No changes to `use-server-table-controls.ts`, `pagination-bar.tsx`, `use-pagination.ts`, `table-sort.ts`, `antd.adapter.tsx`, `destructive-action-confirm-modal.tsx`, or `AuditLogTable.tsx`.

---

## GitHub comments (Open findings)

### 1. `src/pages/videoLibrary/dashboard/components/video-library-columns.tsx` line 49

**Medium (scope):** This PR adds red styling to the video-library Delete menu label — unrelated to the upload event-picker refactor. Please revert or move to a separate PR so SKSH-326 stays scoped to `events/**` and `videos/**` event-select changes.

### 2. `src/pages/events/utils/event.utils.ts` line 382

**Medium (KISS):** `buildUploadEventSelectOptions` is now a one-line pass-through to `mapAthleteRegisteredEventsToSelectOptions`. Please inline at the hook call site and remove the wrapper (or drop the extra hop and call `mapAthleteRegisteredEventsToSelectOptions` directly from `useUploadEventSelectOptions`).

### 3. `src/pages/events/types.ts` line 189

**Medium (Contract):** Updated comments imply role-filtered results (“registered events for athlete/parent” vs “created events for coach”), but `GET /v1/events/me` returns **all active platform events** for every role (`listEventsForViewer` → `findAllActiveEvents`). Please align JSDoc with backend swagger: “all active platform events for the upload event picker.”

---

---
Unrelated video-library Delete label styling (scope)

Risk Level: MEDIUM  
File Path: src/pages/videoLibrary/dashboard/components/video-library-columns.tsx  
Lines: 49

Description:
The only change in the video-library dashboard is wrapping the Delete dropdown label in `<span className="text-red-500!">`. This ticket refactors upload/edit event selection (`useUploadEventSelectOptions`, `event.utils.ts`, `eventService.ts`) and does not touch video-library list behavior.

Impact:
- Unrelated surface bundled into the PR; reviewers/QA must validate video-library action menu styling outside event-picker scope.
- Increases rollback risk if the event-picker change is held.

Recommendation:
Revert the `video-library-columns.tsx` change on this branch or split it into a separate styling PR. Keep SKSH-326 limited to event-select unification under `src/pages/events/**` and `src/pages/videos/**`.

**PR comment (`video-library-columns.tsx` line 49):**  
**Medium:** Delete label red styling is unrelated to the event-picker refactor — please revert here or move to a separate PR.

**Status:** Open

---

---
`buildUploadEventSelectOptions` is a pass-through wrapper (KISS)

Risk Level: MEDIUM  
File Path: src/pages/events/utils/event.utils.ts  
Lines: 382-384  
File Path: src/pages/videos/hooks/useUploadEventSelectOptions.ts  
Lines: 6, 24

Description:
**KISS:** After removing the coach/athlete branch and `mapVideoLibraryEventsToSelectOptions`, `buildUploadEventSelectOptions` only delegates to `mapAthleteRegisteredEventsToSelectOptions` with no additional logic. The hook imports both symbols but needs only one mapping function.

Impact:
- Extra indirection obscures the data flow (API rows → select options) without reuse benefit.
- Future edits may update one function and forget the wrapper.

Recommendation:
Call `mapAthleteRegisteredEventsToSelectOptions` directly from `useUploadEventSelectOptions` and delete `buildUploadEventSelectOptions`:

```ts
// useUploadEventSelectOptions.ts
import { mapAthleteRegisteredEventsToSelectOptions } from "@/pages/events/utils/event.utils";

const eventSelectOptions = useMemo(
  (): EventSelectOption[] => mapAthleteRegisteredEventsToSelectOptions(myEvents),
  [myEvents],
);
```

**PR comment (`event.utils.ts` line 382):**  
**Medium (KISS):** `buildUploadEventSelectOptions` is now a one-line pass-through — please inline `mapAthleteRegisteredEventsToSelectOptions` in the hook and remove the wrapper.

**Status:** Open

---

---
Event-picker comments imply role-filtered API; backend returns all active events (Contract)

Risk Level: MEDIUM  
File Path: src/pages/events/types.ts  
Lines: 189  
File Path: src/api/services/eventService.ts  
Lines: 49  
File Path: src/pages/videos/hooks/useUploadEventSelectOptions.ts  
Lines: 10

Description:
**Contract:** This PR updates several comments to describe `GET /v1/events/me` as returning “registered events (athlete/parent) or created events (coach).” Backend swagger and implementation return **all active platform events** for any authenticated uploader — `listEventsForViewer` ignores role and calls `eventRepository.findAllActiveEvents()`. Coaches previously used `GET /v1/video-library/lookups`; athletes already used `/v1/events/me`. Both sources expose active event slugs; the unification is correct, but the new docs overstate role-specific filtering.

Impact:
- Misleading JSDoc may cause a future developer to reintroduce dual fetch paths or expect registration-scoped lists for athletes.
- Comment drift from the actual API contract complicates cross-repo alignment.

Recommendation:
Align comments with backend swagger summary (“all active platform events for the upload event picker”), e.g.:

```ts
/** Row from `GET /v1/events/me` — active platform events for the upload event picker. */
export type AthleteRegisteredEventItem = { ... };
```

Apply the same wording in `eventService.ts`, `constants.ts`, `useUploadEventSelectOptions.ts`, and `video.types.ts`.

**PR comment (`types.ts` line 189):**  
**Medium (Contract):** Comments suggest role-filtered event lists, but `/v1/events/me` returns all active platform events for every role — please align JSDoc with backend swagger.

**Status:** Open

---

## Positive notes

- **DRY:** Removes the parallel coach (`getVideoLibraryLookups`) vs athlete (`listMyRegisteredEvents`) fetch split; one query and one mapper path for upload/edit flows.
- **Simpler hook:** `useUploadEventSelectOptions` drops role-gated `enabled` flags and a second `useQuery`; `isCoach` remains correctly exported for team/upload behavior in consumers.
- **Cleanup:** Removes unused `BuildUploadEventSelectOptionsParams`, `mapVideoLibraryEventsToSelectOptions`, and the `VideoLibraryLookupOptionDto` import from event utils.
- **Value shape unchanged:** Both old sources used event **slug** as select `value`; `mapAthleteRegisteredEventsToSelectOptions` preserves `{ label: eventName, value: slug }`.
- **Protected modules:** No edits to frozen table/pagination/destructive-modal infrastructure.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Unrelated video-library Delete label styling (scope) | MEDIUM | Open | src/pages/videoLibrary/dashboard/components/video-library-columns.tsx | 49 |
| 2 | `buildUploadEventSelectOptions` is a pass-through wrapper (KISS) | MEDIUM | Open | src/pages/events/utils/event.utils.ts | 382-384 |
| 3 | Event-picker comments imply role-filtered API; backend returns all active events (Contract) | MEDIUM | Open | src/pages/events/types.ts | 189 |

**Merge readiness:** No Critical or High blockers. Three open Medium items (scope, KISS wrapper, API comment accuracy) — acceptable to merge with acknowledgment, or address in follow-up on this branch.
