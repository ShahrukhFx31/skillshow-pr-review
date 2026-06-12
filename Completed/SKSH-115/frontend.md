# Frontend PR Review — skillshow-admin-ui (`SKSH-115`)

**Repo:** skillshow-admin-ui  
**Branch:** `sksh-115`  
**Base:** `main...HEAD` @ `361f361e`  
**Re-verified:** `361f361e` (`fix: feedbacks`)  
**Scope:** Admin dashboard page, dashboard table previews, edit-request table reuse (Critical / High / Medium)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 5 in scope (0 Critical, 2 High, 3 Medium) — **4 fixed**, **1 accepted**, **0 open**

### Protected modules

| Module | Status |
|--------|--------|
| All frozen modules (`use-pagination`, `use-server-table-controls`, audit-log stack, etc.) | **Not modified** ✅ |

### Files reviewed (26 changed)

Admin dashboard (`src/pages/dashboard/admin/*`), shared `DashboardPreviewTableCard`, `dashboard-scroll-table.css`, `use-admin-edit-request-column-handlers.ts`, events preview card, edit-request table preview flags, super-admin table card re-export.

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Reuse `AdminEditRequestTable` on admin dashboard | ✅ Preview via `hidePagination` / `hideSelection` / `hideSorting` |
| Shared column handlers | ✅ `useAdminEditRequestColumnHandlers` extracted and reused |
| Events preview card | ✅ Composes `DashboardPreviewTableCard` |
| Shared table cell className | ✅ `DASHBOARD_TABLE_CELL_CLASSNAME` in `dashboard/constant` |
| Scrollbar styling | ✅ Colocated `dashboard-scroll-table.css` (not `global.css`) |
| Super-admin live ops scroll/mobile | ✅ Accepted — admin-only scope; file not in diff |

### Positive notes

- **DRY:** `SuperAdminDashboardTableCard` is now a re-export of `DashboardPreviewTableCard` — single implementation for dashboard preview tables.
- **DRY:** `resolveEditorAssignmentBackendStatus` centralizes status mapping in the shared column-handlers hook.
- **Welcome copy:** `DASHBOARD_WELCOME_DESCRIPTION_BY_ROLE` and `DASHBOARD_ROLE_PRIORITY` include `Admin`.

---

## GitHub comments

No open Critical or High findings — prior inline comments resolved on `361f361e`.

---

## Findings

---
Dashboard hook duplicates edit-request column handler logic

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: src/pages/dashboard/admin/hooks/use-admin-dashboard-edit-requests.ts  
Lines: 22

Description:
**DRY.** Initial diff duplicated payment/due-date/editor handlers from `adminEditRequest/index.tsx`.

**Re-verification (`361f361e`):** ✅ **Fixed** — `use-admin-dashboard-edit-requests.ts` delegates to `useAdminEditRequestColumnHandlers(listQueryKey)`; shared hook in `use-admin-edit-request-column-handlers.ts`; list page updated to use the same hook.

**PR comment:** **Resolved** — shared `useAdminEditRequestColumnHandlers` extracted.

---

---
EventsDashboardPreviewCard duplicates SuperAdminDashboardTableCard

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: src/pages/events/dashboard/components/events-dashboard-preview-card.tsx  
Lines: 27-38

Description:
**DRY / Global consistency.** Initial diff introduced a parallel preview card implementation.

**Re-verification (`361f361e`):** ✅ **Fixed** — `EventsDashboardPreviewCard` composes `DashboardPreviewTableCard` with event columns and `EventsTableResponsive` as `renderMobile`. `SuperAdminDashboardTableCard` re-exports the same base component.

**PR comment:** **Resolved** — shared `DashboardPreviewTableCard` in use.

---

---
DASHBOARD_TABLE_CELL_CLASSNAME duplicated across dashboard table cards

Risk Level: MEDIUM  
**Status:** ✅ Fixed  
File Path: src/pages/dashboard/constant/index.tsx  
Lines: 41-42

Description:
**DRY.** Identical Ant table cell class string was copy-pasted across preview cards.

**Re-verification (`361f361e`):** ✅ **Fixed** — `DASHBOARD_TABLE_CELL_CLASSNAME` exported from `dashboard/constant/index.tsx`; imported by `dashboard-preview-table-card.tsx`.

**PR comment:** **Resolved** — className hoisted to dashboard constants.

---

---
Dashboard scrollbar styles added to global.css

Risk Level: MEDIUM  
**Status:** ✅ Fixed  
File Path: src/pages/dashboard/components/dashboard-scroll-table.css  
Lines: 1-30

Description:
**File structure.** Initial diff added `.dashboard-scroll-table` rules to `global.css`.

**Re-verification (`361f361e`):** ✅ **Fixed** — styles moved to colocated `dashboard-scroll-table.css`, imported by `dashboard-preview-table-card.tsx`. `global.css` no longer in PR diff.

**PR comment:** **Resolved** — scrollbar styles colocated with dashboard components.

---

---
Super-admin live-ops tables not upgraded to scroll/mobile pattern

**Status: Accepted** — SKSH-115 scope is the **admin** dashboard; `super-admin-live-operations-section.tsx` is **not in this PR diff** (no GitHub inline comment target). Super-admin parity is optional follow-up.

Risk Level: MEDIUM  
File Path: src/pages/dashboard/superAdmin/components/super-admin-live-operations-section.tsx  
Lines: 30, 41 *(unchanged file — not reviewable in GitHub PR UI)*

Description:
**Global consistency (report-only).** Admin events preview adopts scroll/mobile via `DashboardPreviewTableCard`; super-admin live-ops caller unchanged.

Impact:
- Super-admin crew/events preview tables keep pre-PR desktop behavior
- Not a regression

Recommendation:
Optional follow-up: pass `scroll` / `mobileLayout` in `super-admin-live-operations-section.tsx`.

**GitHub comment:** None — file not in PR diff.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Dashboard hook duplicates edit-request column handler logic | HIGH | ✅ Fixed | src/pages/dashboard/admin/hooks/use-admin-dashboard-edit-requests.ts | 22 |
| 2 | EventsDashboardPreviewCard duplicates SuperAdminDashboardTableCard | HIGH | ✅ Fixed | src/pages/events/dashboard/components/events-dashboard-preview-card.tsx | 27-38 |
| 3 | DASHBOARD_TABLE_CELL_CLASSNAME duplicated across dashboard table cards | MEDIUM | ✅ Fixed | src/pages/dashboard/constant/index.tsx | 41-42 |
| 4 | Dashboard scrollbar styles added to global.css | MEDIUM | ✅ Fixed | src/pages/dashboard/components/dashboard-scroll-table.css | 1-30 |
| 5 | Super-admin live-ops tables not upgraded to scroll/mobile pattern | MEDIUM | Accepted | super-admin-live-operations-section.tsx *(not in diff)* | 30 |

**Merge readiness:** No open Critical/High/Medium blockers. All actionable findings fixed on `361f361e`. Ready to merge.
