# Frontend PR Review — skillshow-admin-ui (`SKSH-305`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-305`  
**Base:** `main...HEAD`  
**Scope:** React performance, hooks, JSX/props, Tailwind/file structure (Critical, High, Medium only)  
**Findings:** 1 (0 Critical, 0 High, 1 Medium) — **re-verified: ✅ Fixed**

---

---
List/create routing now depends only on current page activity

Risk Level: MEDIUM
File Path: src/pages/editRequest/index.tsx
Lines: 142, 275-300
Primary PR Comment Line: 276

Description:
The page now uses `hasActiveRequests` (derived from only the currently fetched page) to decide whether to render list mode or create mode. If pagination lands on a page containing only inactive rows while other pages still contain active requests, the UI switches to create flow and hides list navigation context.

Impact:
- Users can be routed into create flow even though active requests still exist elsewhere in their list.
- Existing requests become harder to discover/revisit because list mode is conditionally suppressed by page-local data.

Recommendation:
Drive the list/create decision from a global signal (for example stats API totals for active statuses, or an unpaginated backend flag like `hasActiveRequests`) instead of current-page rows. At minimum, keep list mode visible whenever backend reports any requests in pagination total and gate only CTA behavior by active/inactive classification.

**PR comment (line 276):**  
`hasActiveRequests` is computed from the current paginated page, but it now controls whether we render list mode at all. That means landing on a page with only cancelled/rejected rows can route the user into create flow even if active requests exist on other pages. Could we base this switch on a global backend signal (stats/flag) rather than page-local rows?

**Re-verification:** ✅ Fixed — list/create routing now uses `hasAnyRequests = (listData?.pagination?.total ?? 0) > 0` (line 142), so list mode stays visible for any account with requests regardless of current page contents. Search path always renders list mode with current `uiRequests` (lines 287-292). Create flow only when `!hasAnyRequests && !searchName.trim()` (line 300). `hasActiveEditRequests` remains in eligibility utils for detail/shell logic only, not list routing.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | List/create routing now depends only on current page activity | MEDIUM | ✅ Fixed | `src/pages/editRequest/index.tsx` | 142, 275-300 |

**Merge readiness:** No open Critical/High/Medium blockers.
