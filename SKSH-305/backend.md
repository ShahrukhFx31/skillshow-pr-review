---
Deleting a library video leaves orphaned active edit requests

Risk Level: HIGH
File Path: src/services/edit-request.service.ts
Lines: 108-121
Primary PR Comment Line: 117

Description:
When a user deletes a source video from the library, `detachVideoFromUserEditRequests()` removes the video reference but does not transition lifecycle state when the request is left with zero source videos. This creates requests that remain in actionable statuses (`change_requested`, `change_request_submitted`, etc.) even though they no longer have any source media to resolve.

Impact:
- Admin and user lists can keep showing an edit request as active/workable while it is no longer fulfillable.
- Lifecycle/reporting status becomes inconsistent depending on whether the last source video was removed through request actions (`cancelled`) or through library deletion (stays non-cancelled).

Recommendation:
After detaching a source video, if `requestedVideo.videos` becomes empty, set the request status to `cancelled`, clear fields that depend on active workflow (for example `adminAcceptedAt`), and append a consistent history event for auditability. Reuse the same terminal-state behavior already implemented in source-removal flow to keep state transitions deterministic.
---

PR comment (inline, ready to paste):
When we detach a deleted library video from edit requests, we only mutate the source array and save. If this was the last source slot, the request can remain in an active status with no resolvable media, which creates inconsistent lifecycle behavior versus the `remove_source_video` path (that now cancels). Can we align this path to transition to `cancelled` (and record history) when no source videos remain?

## Summary
| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Deleting a library video leaves orphaned active edit requests | HIGH | Open | `src/services/edit-request.service.ts` | 108-121 |

**Merge readiness:** Open High backend blocker remains.
