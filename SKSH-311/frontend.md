# Frontend PR Review — skillshow-admin-ui (`SKSH-311`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-311`  
**Base:** `main...HEAD`  
**Re-verified:** 2026-06-02 (full pr-review) — @ `cbfc7f92`  
**Scope reviewed:** All **51 files** in `git diff main...HEAD` — events dashboard server table, My Videos dashboard server table, shared hooks/utils.  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency enforced)

**Aligned with:** [backend.md](./backend.md)

### Full review coverage

| Area | Verdict |
|------|---------|
| Events dashboard (`index`, `events-table`, `event-columns`, `use-event-actions`) | ✅ OK |
| My Videos dashboard + linked athlete | ✅ OK |
| Removed client fetch (`fetch-events-list`, `VideoTabsTable`, `event-table-sort`) | ✅ Server-side parity |
| Shared patterns (`useServerTableControls`, `PaginationBar`, `applyServerSort`) | ✅ **Global consistency** across events + my-videos |

### pr-review counts

| Metric | Count |
|--------|-------|
| New findings added | 0 |
| Prior Open → Fixed this pass | 1 |
| Remaining Open | 0 |

---

## GitHub comments (Open findings)

None.

---

---
Linked athlete My Videos always sends `source=athlete`

Risk Level: HIGH  
File Path: src/pages/videos/my-videos/dashboard/utils.ts  
Lines: 56-65

Description (original):
Linked-athlete Media Vault always sent `source: athlete`, excluding `inVideoLibrary` videos.

Impact:
- Skill Show–assigned videos hidden from Media Vault vs pre-migration behavior.

Recommendation:
Omit `source` when `includeSourceTab === false`.

**Re-review:** ✅ **Fixed** — `includeSourceTab = !isLinkedAthleteView`; meta/list queries use `sourceScopeKey: "all"`.

**PR comment:** Resolved on branch.

**Status:** ✅ Fixed

---

---
Dead duplicate `getEventTypeFilterOptions` (DRY)

Risk Level: MEDIUM  
File Path: src/pages/events/dashboard/utils.ts  
Lines: 60-66 (removed)

Description (original):
`getEventTypeFilterOptions()` duplicated `EVENT_TYPE_OPTIONS` and was unused.

Impact:
- **DRY** — parallel filter option definitions could drift.

Recommendation:
Remove dead helper; use `EVENT_TYPE_OPTIONS` only.

**Re-review:** ✅ **Fixed** — function removed; dashboard uses `EVENT_TYPE_OPTIONS` in `index.tsx`.

**Status:** ✅ Fixed

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Linked athlete list always sends `source=athlete` | HIGH | ✅ Fixed | src/pages/videos/my-videos/dashboard/utils.ts | 56-65 |
| 2 | Dead duplicate `getEventTypeFilterOptions` | MEDIUM | ✅ Fixed | src/pages/events/dashboard/utils.ts | removed |

## Positive notes (DRY / KISS / Global consistency)

- **Global consistency:** Events and My Videos both use `useServerTableControls`, meta query (`DEFAULT_LIST_QUERY`), `PaginationBar`, `applyServerSort`.
- **DRY:** CSV export colocated in `events/dashboard/utils.ts`; column defs in `event-columns.tsx` / `my-videos-columns.tsx`; removed duplicate fetch helpers.
- **KISS:** Deleted `VideoTabsTable` full-catalog fetch + client pagination; single hook per feature (`use-my-videos-dashboard-list`, events list in `index.tsx` + `use-event-actions`).

**Merge readiness:** ✅ Clear — **0 Open** Critical/High/Medium. Backend aligned ([backend.md](./backend.md)).
