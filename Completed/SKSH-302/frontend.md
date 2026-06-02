# Frontend PR Review — skillshow-admin-ui (`SKSH-302`)

**Repo:** skillshow-admin-ui  
**Branch:** `sksh-302`  
**Base:** `main...HEAD`  
**Final check:** `412260e1` (`fix: update UI`)  
**Scope:** Notification drawer — unread-only list, clear-all, badge cap, empty state, pagination clamp (Critical / High / Medium only)

**SKSH-302 notification files:** `NotificationContext.tsx`, `notice.tsx`, `notification.types.ts`

---

## Final verification (`412260e1`)

| # | Finding | Status | Evidence |
|---|---------|--------|----------|
| 1 | Clear-all scope vs unread-only (confirm/copy) | **Accepted** | No modal by dev decision; `Clear all` link label (`notice.tsx` 119–132) |
| 2 | In-flight guard on clear | **✅ Fixed** | `isClearingAll`, `disabled`, `aria-busy` on clear button |
| 3 | Badge `cn()` | **✅ Fixed** | `cn()` at lines 101–107 |
| 4 | Mark all as read removed from drawer | **Accepted** | Intentional UI simplification in `412260e1` — per-item read + Clear all only |

**Merge-ready (notification scope):** **Yes** — all findings resolved (2 fixed, 2 accepted).

---

## Re-review summary (original findings)

| # | Original finding | Verdict |
|---|------------------|---------|
| 1 | Clear-all scope vs unread-only drawer | **Accepted** — dev: no confirmation modal; full delete intentional |
| 2 | No confirmation or in-flight guard | **✅ Fixed** — pending/disabled on clear; confirm not required |
| 3 | Badge `className` without `cn()` | **✅ Fixed** — `c96ac627` |
| 4 | Mark all as read removed from drawer | **Accepted** — intentional; drawer uses per-item read + Clear all |

---

---
Clear-all removes full history while drawer shows unread only

Risk Level: HIGH
File Path: src/layouts/components/notice.tsx
Lines: 119-132

Description:
The list uses `unreadOnly: true`; clear calls `DELETE /v1/notifications/clear-all` (full inbox). No confirmation modal per product decision.

Impact:
- Users may not realize read history is also deleted (accepted tradeoff)

Recommendation:
Optional helper copy only if product changes mind; not blocking per dev response.

**Re-review:** **Accepted** — dev: `disabled` while pending is sufficient; no confirmation modal.

---

---
Destructive clear-all has no confirmation or in-flight guard

Risk Level: MEDIUM
File Path: src/layouts/components/notice.tsx
Lines: 119-132

Description:
`isClearingAll` wired with `disabled` and `aria-busy` on the `Clear all` link `Button`. No confirmation gate (declined by dev).

Impact:
- Double-submit prevented during mutation

Recommendation:
N/A — in-flight requirement met.

**Re-review:** **✅ Fixed** at `412260e1` (link-style `Button` with `disabled={isClearingAll}`).

---

---
Badge count uses duplicated conditional class strings without `cn()`

Risk Level: MEDIUM
File Path: src/layouts/components/notice.tsx
Lines: 101-107

Description:
Badge uses `cn()` for conditional layout classes and `99+` cap.

**Re-review:** **✅ Fixed**.

---

---
Mark all as read removed from notification drawer (new in `412260e1`)

Risk Level: MEDIUM
File Path: src/layouts/components/notice.tsx
Lines: 48-58, 117-133

Description:
`fix: update UI` (`412260e1`) removed the header “Mark all as read” control and stopped destructuring `markAllAsRead` from `useNotifications()`. `markAllAsRead` remains on `NotificationContext` but has **no UI consumer** in the repo. Users can only mark items one-by-one or use destructive “Clear all.”

Impact:
- No bulk mark-as-read for users with many unread notifications
- Likely unintentional regression when simplifying the header to a single “Clear all” link

Recommendation:
Restore mark-all-as-read alongside clear (e.g. link or icon next to “Clear all”), or confirm removal with product and delete unused `markAllAsRead` from context/types:

```tsx
const { ..., markAllAsRead, clearAll, isClearingAll } = useNotifications();
// Header: Mark all as read + Clear all
```

**PR comment (line 118):** **Medium:** `412260e1` removed “Mark all as read” from the drawer; only per-item read + Clear all remain—please restore bulk mark-as-read or confirm intentional removal.

**Re-review:** **Accepted** — Removal is intentional product/UI choice (`412260e1`): header is “Clear all” only; users mark individual items as read.

---

## Positive notes

- `unreadOnly: true` + “No new notifications” empty state
- Page clamp when unread pages shrink
- `99+` badge cap with `cn()`
- Clear-all uses `@/ui` `Button` link variant with pending guard

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Clear-all removes full history while drawer shows unread only | HIGH | Accepted | src/layouts/components/notice.tsx | 119-132 |
| 2 | Destructive clear-all has no confirmation or in-flight guard | MEDIUM | ✅ Fixed | src/layouts/components/notice.tsx | 119-132 |
| 3 | Badge count uses duplicated conditional class strings without `cn()` | MEDIUM | ✅ Fixed | src/layouts/components/notice.tsx | 101-107 |
| 4 | Mark all as read removed from notification drawer | MEDIUM | Accepted | src/layouts/components/notice.tsx | 48-58, 117-133 |
