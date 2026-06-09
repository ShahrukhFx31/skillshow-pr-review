# Backend PR Review — skillshow (`SKSH-327`)

**Repo:** skillshow  
**Branch:** `sksh-327`  
**Base:** `main...HEAD` @ `6988c13`  
**Initial review:** 2026-06-09  
**Scope:** Align linked-athlete play-url access with view-only linked video contract (`getLinkedVideo`); remove redundant `findLinkedVideoKey` helper (Critical / High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 0 (0 Critical, 0 High) — **0 Open**

### Protected modules

| Module | Status |
|--------|--------|
| `list-query.validation.ts`, `list-query-aggregation.utils.ts`, `list-row-repository.utils.ts`, `mongo-change-stream.service.ts`, `audit-log.utils.ts`, `change-stream.utils.ts` | **Not modified** |

No list endpoints, bulk row ops, audit logs, or change streams in this diff.

### Files reviewed

| File | Change |
|------|--------|
| `src/services/athlete.service.ts` | `getLinkedEncryptedPlayUrl` uses view access path; `findLinkedVideoKey` removed |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Removed single-use `findLinkedVideoKey` pass-through | ✅ KISS — logic inlined at sole call site |
| Play-url access matches `getLinkedVideo` (relation middleware + public video) | ✅ Contract alignment |
| `patchLinkedVideo` / `findLinkedVideoForUpdate` still use `assertLinkedVideoShareAccess` for mutations | ✅ View vs manage separation preserved |
| `assertLinkedViewerMayAccessVideo` applied on play-url path | ✅ Private uploads hidden |
| Protected modules untouched | ✅ |
| Global list/bulk patterns | N/A — single-service method change |

### Positive notes

- **Contract:** Previously `getLinkedEncryptedPlayUrl` routed through `findLinkedVideoForUpdate` → `assertLinkedVideoShareAccess`, which requires `SHARE_VIDEOS` parent permission. That blocked coaches/parents who can **view** linked public videos (`getLinkedVideo`, `listLinkedVideos`) from obtaining a play URL. The new path matches `getLinkedVideo`: `linkedAthleteRelationGuards()` middleware + `resolveOwnerUserIdForAthlete` + `findOwnedVideoForUpdate` + `assertLinkedViewerMayAccessVideo`.
- **KISS:** Removing `findLinkedVideoKey` eliminates an unnecessary wrapper that only forwarded to `findLinkedVideoForUpdate` and extracted `doc.key`.
- **Cross-stack:** Frontend `VideoPlayer` now passes `relationId` and calls `getPlayUrlForContext` → `GET /athletes/:relationId/videos/:videoId/play-url`; backend change unblocks that flow.

---

## GitHub comments

No Critical or High findings — no inline comments required.

---

## Findings

No Critical or High findings.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| — | No Critical or High findings | — | — | — | — |

**Merge readiness:** No open Critical/High blockers. Backend change is a focused contract fix aligned with `getLinkedVideo`; safe to merge from a backend review perspective. Consider adding a unit test mirroring `getLinkedVideo hides private videos from parent/coach` for `getLinkedEncryptedPlayUrl` in a follow-up (optional hardening, not a merge blocker).
