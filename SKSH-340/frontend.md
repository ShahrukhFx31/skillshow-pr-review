# Frontend PR Review — skillshow-admin-ui (`SKSH-340`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-340`  
**Base:** `main...HEAD` @ `7477ebe3`  
**Scope:** Events dashboard event-type filter refactor; event view related section Edit/Done toggle; move related section outside `Form`; incidental account tabs scrollbar class change  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 4 in scope (0 Critical, 2 High, 2 Medium) — **2 Open**, **2 Accepted**

### Protected modules

| Module | Status |
|--------|--------|
| `use-pagination.ts`, `pagination-bar.tsx`, `use-server-table-controls.ts`, `destructive-action-confirm-modal.tsx`, audit-log stack, `antd.adapter.tsx` | **Not modified** ✅ |

`DestructiveActionConfirmModal` still used for athlete/crew removal ✅  
`useServerTableControls` + `filterKey` still wires `eventTypeFilter` / `listingTab` for page reset ✅

### Files reviewed (8 changed)

| File | Change |
|------|--------|
| `events/dashboard/components/events-actions.tsx` | Event type Select: placeholder + `allowClear`, remove synthetic "All events" option |
| `events/dashboard/index.tsx` | `eventTypeFilter` default `""` instead of `null` |
| `events/dashboard/types.ts` | Filter type `EventType \| ""` |
| `events/onboarding/components/event-view/event-view-related-section.tsx` | Local Edit/Done toggle; remove parent `readOnly` |
| `events/onboarding/constants.ts` | Edit/Done label constants |
| `events/onboarding/index.tsx` | Related section outside `Form`; drop `readOnly` prop |
| `events/onboarding/types.ts` | Remove `readOnly` from section props |
| `user/account/index.tsx` | `scrollbar-none` class (unrelated) |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Event type filter uses `""` sentinel + `allowClear` instead of duplicate "All events" option | ✅ KISS |
| `eventTypeFilter` in `extraFilterState` → `filterKey` page reset | ✅ Contract |
| `queryKey` includes `eventTypeFilter`; API uses `eventTypeFilter \|\| undefined` | ✅ Contract |
| Edit/Done labels in feature `constants.ts` | ✅ |
| Related section moved outside disabled `Form` (tables were never form fields) | ✅ |
| `DestructiveActionConfirmModal` for removals | ✅ Contract |
| Done toggle unmounts modals without clearing open/pending state | ⚠️ See #1 |
| Event **edit** route now requires extra Edit click for roster mutations | ✅ Accepted (#2) — intentional Edit/Done on all routes |
| Account tabs `scrollbar-none` vs established `scrollbar-hide` / `[scrollbar-width:none]` | ⚠️ See #3 |
| Unrelated account change bundled in events ticket | ✅ Accepted (#4) |

---

## GitHub comments (Open findings)

### 1. `src/pages/events/onboarding/components/event-view/event-view-related-section.tsx` line 185

**PR comment (line 185):** **High:** Clicking Done unmounts add/remove modals but leaves `addAthleteOpen`, `addCrewOpen`, `athleteToRemove`, and `crewToRemove` set. Re-entering Edit can immediately reopen a modal or confirm dialog. Reset those four states when `isTablesEditing` becomes `false` (in the toggle handler or a `useEffect`).

### 2. `src/pages/user/account/index.tsx` line 115

**PR comment (line 115):** **Medium:** `scrollbar-none` is not a defined utility in this repo (`scrollbar-hide` is in `global.css`; siblings use `[scrollbar-width:none]`). This likely stops hiding the Firefox scrollbar on account tabs. Revert to `[scrollbar-width:none]` or use `scrollbar-hide`.

---

## Findings

---
Done toggle leaves stale modal and confirm state

Risk Level: HIGH  
File Path: src/pages/events/onboarding/components/event-view/event-view-related-section.tsx  
Lines: 54-59, 185-187, 273-321

Description:
**KISS / behavioral bug.** `isTablesEditing` gates rendering of `DestructiveActionConfirmModal`, `EventViewAddAthleteModal`, and `EventViewAddCrewModal` via `{!tablesReadOnly && (…)}`. Toggling Done sets `tablesReadOnly` to `true` and unmounts those components, but `addAthleteOpen`, `addCrewOpen`, `athleteToRemove`, and `crewToRemove` are not cleared. Clicking Edit again remounts modals with stale `open` state.

Impact:
- User clicks Done with an add modal open → Edit again → modal reappears unexpectedly
- User opens remove confirm, clicks Done → Edit again → destructive confirm reopens without a new remove action
- Confusing, error-prone roster management on event view/edit pages

Recommendation:
When exiting edit mode, reset dependent UI state:

```tsx
const exitTablesEditing = useCallback(() => {
  setIsTablesEditing(false);
  setAddAthleteOpen(false);
  setAddCrewOpen(false);
  setAthleteToRemove(null);
  setCrewToRemove(null);
}, []);

// Done button:
onClick={() => (isTablesEditing ? exitTablesEditing() : setIsTablesEditing(true))}
```

**PR comment (line 185):** **High:** Reset add/remove modal state when Done is clicked so Edit does not resurrect stale dialogs.

---

---
Event edit route regresses to two-step roster editing

**Status: Accepted** — intentional: roster tables use the same Edit/Done gate on view and edit routes for consistent, explicit edit mode.

Risk Level: HIGH  
File Path: src/pages/events/onboarding/components/event-view/event-view-related-section.tsx  
Lines: 54-55

Description:
**Global consistency / UX regression.** The PR removes `readOnly={isView}` from the parent. Previously roster tables were editable immediately on the event **edit** route (`readOnly=false`) and read-only on **view**. `isTablesEditing` now defaults to `false` on both routes, so users on `/events/:id/edit` must click section Edit before Add/Remove crew or athletes — an extra step that did not exist before.

Impact:
- Extra friction on the primary event edit workflow
- Inconsistent with user expectation that "edit event" enables all editable surfaces

Recommendation:
No code change required. Accepted as deliberate UX: form edit and roster edit remain separate, with explicit Edit/Done on the related section across routes.

**GitHub comment:** None required — accepted as intentional.

---

---
Invalid scrollbar utility on account tabs

Risk Level: MEDIUM  
File Path: src/pages/user/account/index.tsx  
Lines: 115

Description:
**KISS / styling contract.** The diff replaces working `[scrollbar-width:none]` with `scrollbar-none`. This project defines `.scrollbar-hide` in `global.css` and uses `[scrollbar-width:none]` + webkit overrides on sibling tab bars (`myAthlete/viewProfile`, `share-account-tab`). `scrollbar-none` is not configured in `tailwind.config.ts` and is not the established hide-scroll pattern.

Impact:
- Horizontal tab bar may show a visible Firefox scrollbar on account page (regression vs prior markup)
- Inconsistent with other tab-scroll containers in the app

Recommendation:
Revert to the prior pattern or use the project utility:

```tsx
className="... scrollbar-hide [-ms-overflow-style:none] [&::-webkit-scrollbar]:hidden ..."
```

**PR comment (line 115):** **Medium:** Use `scrollbar-hide` or `[scrollbar-width:none]` — `scrollbar-none` is not a defined utility here.

---

---
Unrelated account change in events ticket PR

**Status: Accepted** — intentional to ship the account tabs scrollbar tweak with this branch; scope mixing acknowledged.

Risk Level: MEDIUM  
File Path: src/pages/user/account/index.tsx  
Lines: 115

Description:
**Scope.** The account tabs scrollbar class change is unrelated to events dashboard filter handling or event view related-section UI (the stated commit scope). It increases review surface and risks an accidental scrollbar regression on a different feature.

Impact:
- Harder to bisect or revert events-only changes
- Unrelated UX risk mixed into SKSH-340

Recommendation:
No code change required. Revert or split only if the team later wants stricter PR scope hygiene.

**GitHub comment:** None required — accepted as intentional.

---

## Positive notes

- Event type filter refactor is cleaner: `""` sentinel, Select `placeholder` + `allowClear`, no duplicate "All events" option row (**DRY**).
- `eventTypeFilter` remains in `useServerTableControls` `extraFilterState` — page resets on filter/tab change (**Contract**).
- Moving `EventViewRelatedSection` outside the disabled `Form` is correct; roster tables are independent of form field `disabled` context.
- View-page roster editing via section Edit/Done (without navigating to form edit) is a sensible UX split from top-level "Edit Event".
- Label constants `EVENT_VIEW_RELATED_SECTION_EDIT_LABEL` / `DONE_LABEL` follow feature constants placement.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Done toggle leaves stale modal and confirm state | HIGH | Open | src/pages/events/onboarding/components/event-view/event-view-related-section.tsx | 54-59, 185-187, 273-321 |
| 2 | Event edit route regresses to two-step roster editing | HIGH | Accepted | src/pages/events/onboarding/components/event-view/event-view-related-section.tsx | 54-55 |
| 3 | Invalid scrollbar utility on account tabs | MEDIUM | Open | src/pages/user/account/index.tsx | 115 |
| 4 | Unrelated account change in events ticket PR | MEDIUM | Accepted | src/pages/user/account/index.tsx | 115 |

**Merge readiness:** **Not merge-ready** — fix High #1 (stale modal state) before merge; address Medium #3 (scrollbar utility) or revert to `[scrollbar-width:none]` / `scrollbar-hide`.
