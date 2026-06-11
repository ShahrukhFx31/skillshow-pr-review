# Frontend PR Review — skillshow-admin-ui (`SKSH-115`)

**Repo:** skillshow-admin-ui  
**Branch:** `sksh-115`  
**Base:** `main...HEAD` @ `c7599a3d`  
**Scope:** Admin dashboard page, dashboard table previews, edit-request table reuse, events preview card (Critical / High / Medium)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 5 in scope (0 Critical, 2 High, 3 Medium) — **4 Open**, **1 Accepted**

### Protected modules

| Module | Status |
|--------|--------|
| All frozen modules (`use-pagination`, `use-server-table-controls`, audit-log stack, etc.) | **Not modified** ✅ |

### Files reviewed (23 changed)

Admin dashboard (`src/pages/dashboard/admin/*`), shared dashboard table utilities, super-admin table card enhancements, events preview card/responsive, admin edit-request table column/sorting/selection flags, minor my-videos layout tweak.

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Reuse `AdminEditRequestTable` on admin dashboard | ✅ Good — preview mode via `hidePagination` / `hideSelection` / `hideSorting` |
| Dashboard edit-request column handlers | ⚠️ Duplicated from list page (see #1) |
| Events preview card vs `SuperAdminDashboardTableCard` | ⚠️ Parallel implementations (see #2) |
| Shared table cell className | ⚠️ Copy-pasted (see #3) |
| Scrollbar styling in `global.css` | ⚠️ Feature-specific global rules (see #4) |
| Super-admin live ops vs admin events preview UX | ✅ Accepted — admin-only scope; super-admin file not in diff (see #5) |

---

## GitHub comments (Open findings)

### 1. `src/pages/dashboard/admin/hooks/use-admin-dashboard-edit-requests.ts` line 52

**PR comment (line 52):** **High (DRY):** Payment/due-date/editor handlers (lines 52–98) and `columnHandlers` (lines 100–110) duplicate `adminEditRequest/index.tsx` (~200–265). Extract shared `useAdminEditRequestColumnHandlers(listQueryKey)` so dashboard preview and list page stay in sync.

### 2. `src/pages/events/dashboard/components/events-dashboard-preview-card.tsx` line 22

**PR comment (line 22):** **High (DRY / Global consistency):** `EventsDashboardPreviewCard` mirrors `SuperAdminDashboardTableCard` (Card/Grid/scroll/mobile branch). Compose the shared card with event columns + `EventsTableResponsive`, or extract `DashboardPreviewTableCard` used by both.

### 3. `src/pages/events/dashboard/components/events-dashboard-preview-card.tsx` line 10

**PR comment (line 10):** **Medium (DRY):** `DASHBOARD_TABLE_CELL_CLASSNAME` is duplicated here and in `super-admin-dashboard-table-card.tsx` line 10. Hoist to `src/pages/dashboard/constant/index.tsx` as `DASHBOARD_TABLE_CELL_CLASSNAME`.

### 4. `src/global.css` line 543

**PR comment (line 543):** **Medium:** `.dashboard-scroll-table` scrollbar rules (lines 543–569) are dashboard-specific. Colocate with dashboard table cards (Tailwind/CSS module) instead of `global.css`.

### 5. `src/pages/dashboard/superAdmin/components/super-admin-live-operations-section.tsx` line 30

**PR comment (line 30):** **Medium (Global consistency):** Both `SuperAdminDashboardTableCard` instances (lines 30, 41) omit `scroll` / `mobileLayout` added in this PR. Admin events preview uses them — adopt here or track super-admin parity as follow-up.

---

## Findings

---
Dashboard hook duplicates edit-request column handler logic

Risk Level: HIGH  
File Path: src/pages/dashboard/admin/hooks/use-admin-dashboard-edit-requests.ts  
Lines: 52-120

Description:
**DRY.** `useAdminDashboardEditRequests` copies the payment-status, due-date, and editor-assignment mutation handlers, optimistic cache patches, and `columnHandlers` memo from `adminEditRequest/index.tsx` (lines ~200–265). Only the query key and fetch params differ.

Impact:
- Future fixes to assignment status mapping or cache patching must be applied in two places
- Risk of dashboard preview behaving differently from the full list (already uses slightly different status constants in places)

Recommendation:
Extract shared logic:

```typescript
// e.g. src/pages/adminEditRequest/hooks/use-admin-edit-request-column-handlers.ts
export function useAdminEditRequestColumnHandlers(
  listQueryKey: readonly unknown[],
  options?: { isInternalRevisionView?: boolean },
): AdminEditRequestColumnHandlers { /* shared mutations */ }
```

Use it from both `adminEditRequest/index.tsx` and `use-admin-dashboard-edit-requests.ts`.

**PR comment (line 52):** **High (DRY):** Extract shared `useAdminEditRequestColumnHandlers` — handlers at lines 52–98 duplicate list-page mutation/handler logic.

---

---
EventsDashboardPreviewCard duplicates SuperAdminDashboardTableCard

Risk Level: HIGH  
File Path: src/pages/events/dashboard/components/events-dashboard-preview-card.tsx  
Lines: 22-77

Description:
**DRY / Global consistency.** This PR enhances `SuperAdminDashboardTableCard` with scroll, flex layout, and optional `DashboardTableResponsive`, then introduces `EventsDashboardPreviewCard` with nearly identical Card/Table structure, the same `DASHBOARD_TABLE_CELL_CLASSNAME` string, scroll wiring, and mobile branching — but as a separate component.

Impact:
- Two dashboard preview card implementations to maintain
- Scrollbar/flex fixes may be applied to one card and missed on the other

Recommendation:
Extend `SuperAdminDashboardTableCard` to accept a custom mobile renderer slot, or extract `DashboardPreviewTableCard` under `src/pages/dashboard/components/` used by super-admin tables and events preview. Pass event-specific columns and `EventsTableResponsive` for mobile.

**PR comment (line 22):** **High:** `EventsDashboardPreviewCard` duplicates `SuperAdminDashboardTableCard` — extract a shared dashboard preview card.

---

---
DASHBOARD_TABLE_CELL_CLASSNAME duplicated across dashboard table cards

Risk Level: MEDIUM  
File Path: src/pages/events/dashboard/components/events-dashboard-preview-card.tsx (anchor); `super-admin-dashboard-table-card.tsx` line 10 (duplicate)  
Lines: 10-11

Description:
**DRY.** Identical Ant table header/cell utility class string is defined in both `events-dashboard-preview-card.tsx` (line 10) and `super-admin-dashboard-table-card.tsx` (line 10).

Impact:
- Styling drift if one card is updated and the other is not

Recommendation:
Move to `src/pages/dashboard/constant/index.tsx` (next to `DASHBOARD_LAYOUT`) as `DASHBOARD_TABLE_CELL_CLASSNAME` and import in both cards.

**PR comment (line 10):** **Medium (DRY):** Hoist shared `DASHBOARD_TABLE_CELL_CLASSNAME` to dashboard constants (duplicate at `super-admin-dashboard-table-card.tsx` line 10).

---

---
Dashboard scrollbar styles added to global.css

Risk Level: MEDIUM  
File Path: src/global.css  
Lines: 543-569

Description:
**File structure / styling conventions.** Feature-specific `.dashboard-scroll-table` scrollbar overrides were added to `global.css`. Project convention keeps styling colocated with components; `DASHBOARD_LAYOUT.scrollTableClassName` already references the class.

Impact:
- `global.css` grows with dashboard-only concerns
- Harder to trace styling ownership

Recommendation:
Colocate via Tailwind plugin, CSS module imported by dashboard table cards, or `@layer components` in a dashboard-scoped stylesheet imported from the dashboard route — not app-wide `global.css`.

**PR comment (line 543):** **Medium:** Move `.dashboard-scroll-table` scrollbar rules (lines 543–569) out of `global.css` into dashboard-colocated styles.

---

---
Super-admin live-ops tables not upgraded to scroll/mobile pattern

**Status: Accepted** — SKSH-115 scope is the **admin** dashboard; `super-admin-live-operations-section.tsx` is **not in this PR diff** (no changed lines → no GitHub inline comment target). Super-admin parity is optional follow-up.

Risk Level: MEDIUM  
File Path: src/pages/dashboard/superAdmin/components/super-admin-live-operations-section.tsx  
Lines: 30, 41 *(unchanged file — not reviewable in GitHub PR UI)*

Description:
**Global consistency (report-only).** This PR enhances `SuperAdminDashboardTableCard` and wires scroll/mobile on the **admin** events preview (`EventsDashboardPreviewCard`), but does not touch `SuperAdminLiveOperationsSection`, which still calls the table card without `scroll` or `mobileLayout`.

Impact:
- Super-admin crew/events preview tables keep pre-PR desktop behavior while admin dashboard gets scroll/mobile
- Not a regression — existing super-admin UX unchanged

Recommendation:
Optional follow-up ticket: pass `scroll` / `mobileLayout` in `super-admin-live-operations-section.tsx`. Not required to merge SKSH-115.

**GitHub comment:** None — file not in PR diff; cannot anchor an inline review comment.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Dashboard hook duplicates edit-request column handler logic | HIGH | Open | src/pages/dashboard/admin/hooks/use-admin-dashboard-edit-requests.ts | 52-120 |
| 2 | EventsDashboardPreviewCard duplicates SuperAdminDashboardTableCard | HIGH | Open | src/pages/events/dashboard/components/events-dashboard-preview-card.tsx | 22 |
| 3 | DASHBOARD_TABLE_CELL_CLASSNAME duplicated across dashboard table cards | MEDIUM | Open | events-dashboard-preview-card.tsx / super-admin-dashboard-table-card.tsx | 10 |
| 4 | Dashboard scrollbar styles added to global.css | MEDIUM | Open | src/global.css | 543 |
| 5 | Super-admin live-ops tables not upgraded to scroll/mobile pattern | MEDIUM | Accepted | src/pages/dashboard/superAdmin/components/super-admin-live-operations-section.tsx | 30 *(not in diff)* |

**Merge readiness:** **Not merge-ready** — 2 open High (DRY duplication) and 2 open Medium. No frontend Critical blockers, but backend Critical KPI regression must be fixed first. Address High findings or extract shared hooks/cards before merge.
