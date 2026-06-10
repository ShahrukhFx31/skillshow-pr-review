# Frontend PR Review — skillshow-admin-ui (`SKSH-334`)

**Repo:** skillshow-admin-ui — `https://github.com/fx31labs-mvp/skillshow-admin-ui.git`  
**Branch:** `SKSH-334`  
**Base:** `main...HEAD` @ `d3546798`  
**Initial review:** 2026-06-10  
**Scope:** Link event name and video library name columns (desktop + responsive) to view pages with breadcrumb state (Critical / High / Medium)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [backend.md](./backend.md)

**Findings:** 1 (0 Critical, 0 High, 1 Medium)

### Protected modules

| Module | Status |
|--------|--------|
| `pagination-bar.tsx`, `use-pagination.ts`, `use-server-table-controls.ts`, `table-sort.ts`, `destructive-action-confirm-modal.tsx`, `antd.adapter.tsx`, audit-log components | **Not modified** |

Server list controls, pagination, and sort wiring are unchanged — only column `render` helpers updated.

### Files reviewed

| File | Change |
|------|--------|
| `src/pages/events/dashboard/components/event-columns.tsx` | `renderEventTableName` with `Link` + breadcrumb trail |
| `src/pages/events/dashboard/components/events-table-responsive.tsx` | Reuse `renderEventTableName` in mobile cards |
| `src/pages/videoLibrary/dashboard/components/video-library-columns.tsx` | `renderVideoLibraryTableName` with `Link` + breadcrumb trail |
| `src/pages/videoLibrary/dashboard/components/video-library-table-responsive.tsx` | Reuse `renderVideoLibraryTableName` in mobile cards |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| `renderEventTableName` / `renderVideoLibraryTableName` extracted and shared with responsive tables | ✅ DRY |
| Event links use `buildEventViewPath(slug)` + `buildEventsFormBreadcrumbTrail` (matches onboarding cancel/nav pattern) | ✅ Contract |
| Video library links use `VIDEO_LIBRARY_ROUTES.view` + `videoLibraryViewBreadcrumbTrail` | ✅ Contract |
| Slug-less events fall back to non-link `Typography.Text` with ellipsis | ✅ |
| Protected table/pagination modules untouched | ✅ |
| Event vs video library linked-name ellipsis pattern | ⚠️ See #1 |

### Positive notes

- **DRY:** Both features extract a single render helper consumed by desktop columns and responsive card layouts — avoids copy-paste between breakpoints.
- **Navigation:** Breadcrumb `state` on event name links matches `events/onboarding/index.tsx` view navigation.
- **Video library:** Ellipsis + `title` on linked names handles long titles in constrained columns and mobile cards.

---

## GitHub comments

### `src/pages/events/dashboard/components/event-columns.tsx` line 61

The video library link helper wraps the label in `Typography.Text` with `ellipsis` (`video-library-columns.tsx:43-45`), but this `Link` renders `{record.eventName}` as plain text (`event-columns.tsx:61`). Long event names can overflow the desktop column and mobile card title instead of truncating with a tooltip. Mirror the video-library pattern: nest `Typography.Text` with `ellipsis={{ tooltip: record.eventName }}` inside the `Link`, and add `title={record.eventName}` on the `Link` (`event-columns.tsx:56-60`).

---

## Findings

---
Event name link missing ellipsis wrapper (inconsistent with video library in same PR)

Risk Level: MEDIUM  
File Path: src/pages/events/dashboard/components/event-columns.tsx  
Lines: 54-62

Description:
**DRY / Global consistency.** This PR introduces linked name renderers for both events and video library. `renderVideoLibraryTableName` nests `Typography.Text` with `ellipsis={{ tooltip: record.videoName }}` inside the `Link`, but `renderEventTableName` renders `{record.eventName}` as plain text in the slug branch. The no-slug fallback correctly uses `Typography.Text` ellipsis; the linked branch does not.

Impact:
- Long event names may overflow the 240px desktop column or the `max-w-[min(100%,16rem)]` mobile card title instead of truncating with a tooltip.
- Inconsistent truncation behavior between two tables changed in the same PR.

Recommendation:
Mirror the video-library helper:

```tsx
const viewPath = buildEventViewPath(slug);
return (
  <Link
    className="block! min-w-0 max-w-full cursor-pointer font-medium text-inherit!"
    state={{ breadcrumbTrail: buildEventsFormBreadcrumbTrail("View Event", viewPath) }}
    title={record.eventName}
    to={viewPath}
  >
    <Typography.Text className="block! min-w-0 max-w-full" ellipsis={{ tooltip: record.eventName }}>
      {record.eventName}
    </Typography.Text>
  </Link>
);
```

**PR comment (`event-columns.tsx` line 61):** The video library link helper wraps the label in `Typography.Text` with `ellipsis` (`video-library-columns.tsx:43-45`), but this `Link` renders `{record.eventName}` as plain text. Long event names can overflow the desktop column and mobile card title instead of truncating with a tooltip. Mirror the video-library pattern: nest `Typography.Text` with `ellipsis={{ tooltip: record.eventName }}` inside the `Link`, and add `title={record.eventName}` on the `Link`.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Event name link missing ellipsis wrapper (inconsistent with video library in same PR) | MEDIUM | Open | src/pages/events/dashboard/components/event-columns.tsx | 54-62 |

**Merge readiness:** No Critical/High blockers. One open Medium (event name ellipsis inconsistency) — low risk; acceptable to merge with follow-up if preferred.
