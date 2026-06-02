---
List/create routing now depends only on current page activity

Risk Level: MEDIUM
File Path: src/pages/editRequest/index.tsx
Lines: 279-305
Primary PR Comment Line: 280

Description:
The page now uses `hasActiveRequests` (derived from only the currently fetched page) to decide whether to render list mode or create mode. If pagination lands on a page containing only inactive rows while other pages still contain active requests, the UI switches to create flow and hides list navigation context.

Impact:
- Users can be routed into create flow even though active requests still exist elsewhere in their list.
- Existing requests become harder to discover/revisit because list mode is conditionally suppressed by page-local data.

Recommendation:
Drive the list/create decision from a global signal (for example stats API totals for active statuses, or an unpaginated backend flag like `hasActiveRequests`) instead of current-page rows. At minimum, keep list mode visible whenever backend reports any requests in pagination total and gate only CTA behavior by active/inactive classification.
---

PR comment (inline, ready to paste):
`hasActiveRequests` is computed from the current paginated page, but it now controls whether we render list mode at all. That means landing on a page with only cancelled/rejected rows can route the user into create flow even if active requests exist on other pages. Could we base this switch on a global backend signal (stats/flag) rather than page-local rows?

## Summary
| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | List/create routing now depends only on current page activity | MEDIUM | Open | `src/pages/editRequest/index.tsx` | 279-305 |

**Merge readiness:** Open Medium frontend blocker remains.
