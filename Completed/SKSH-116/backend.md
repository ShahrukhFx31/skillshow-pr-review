# Backend PR Review — skillshow (`SKSH-116`)

**Repo:** skillshow (main API)  
**Branch:** `SKSH-116-main`  
**Base:** `main...HEAD`  
**Re-verified:** `f65ed2b` (HEAD); no edit-request backend changes since `66514da`  
**Scope:** Code introduced or materially changed on this branch (Critical & High only)  
**Findings:** 8 (0 Critical, 8 High) — **5 fixed**, **3 accepted**, **0 open**, **0 new Critical/High**

---

---
Unescaped user input in MongoDB `$regex` filters

Risk Level: HIGH  
File Path: src/services/edit-request.service.ts  
Lines: 200, 227

Description:
`buildListFilter` and `applyAdminSearchFilter` previously passed raw user strings into `$regex` without escaping metacharacters.

Impact:
- ReDoS or expensive collection scans from malicious search input

Recommendation:
Use `containsInsensitive` from `search.utils.ts` for all user-driven regex filters.

**PR comment (line 200):** **High:** Escape user search terms before `$regex` (use `containsInsensitive`) to avoid ReDoS.

**Re-review:** ✅ Fixed — `containsInsensitive` used at ~200 and ~227.

---

---
No ownership check on replace/remove source video

**Status: Accepted** — team decision; not blocking SKSH-116.

Risk Level: HIGH  
File Path: src/services/edit-request.service.ts  
Lines: (replace_video / remove_source_video actions)

Description:
Admin/participant actions can replace or remove a `videoId` without verifying the video belongs to the edit request owner.

Impact:
- Theoretical cross-request video reference if IDs are guessed (accepted product risk)

Recommendation:
None for SKSH-116.

---

---
`/file` stream route skips download audit

Risk Level: HIGH  
File Path: src/routes/edit-request.routes.ts  
Lines: (removed route)

Description:
A direct file-stream admin route bypassed presigned URL and download-audit patterns used elsewhere.

Impact:
- Unaudited video access path for admins

Recommendation:
Remove the route; use presigned download/view URLs only.

**PR comment:** **High:** Remove unaudited `/file` stream — align with presigned + audit flow.

**Re-review:** ✅ Fixed — route and `getAdminVideoFileStreamMeta` removed.

---

---
Admin `submitFeedback` sets version status to `approved`

Risk Level: HIGH  
File Path: src/services/edit-request.service.ts  
Lines: (admin feedback / review status utils)

Description:
Admin feedback incorrectly moved versions to `approved`, blocking the athlete review workflow.

Impact:
- Athletes could not review after admin feedback

Recommendation:
Use `WAITING_FOR_REVIEW` / `ADMIN_FEEDBACK` via `resolveEffectiveReviewStatus`.

**Re-review:** ✅ Fixed — display and effective status handling updated.

---

---
N+1 queries when mapping admin outputs and versions

Risk Level: HIGH  
File Path: src/services/edit-request-output.service.ts  
Lines: 326-361, 843

Description:
Output list/detail mapping previously loaded versions per output in a loop.

Impact:
- Linear DB round-trips as output count grows

Recommendation:
Batch-load versions with `findByEditRequestId` and `mapAdminOutputsWithBatchLoad`.

**Re-review:** ✅ Fixed — batch load in place.

---

---
Socket join loads all admin user IDs

**Status: Accepted** — team decision; not blocking SKSH-116.

Risk Level: HIGH  
File Path: src/services/edit-request.service.ts  
Lines: (socket join / `canActorAccessEditRequest`)

Description:
Joining an edit-request room resolves access via `findActiveUserIdsWithPermissionRead`, loading all admin IDs.

Impact:
- Extra query cost on join (accepted for correct room membership)

Recommendation:
None for SKSH-116.

---

---
Unbounded user `$in` for admin list search

Risk Level: HIGH  
File Path: src/repositories/user.repository.ts  
Lines: (admin search match cap)

Description:
Admin edit-request search could build an unbounded `$in` list of user IDs from regex user lookup.

Impact:
- Large `$in` arrays and slow list queries

Recommendation:
Cap matches with `USER_SEARCH_MAX_MATCHES` (200) and `capSearchMatchIds`.

**Re-review:** ✅ Fixed.

---

---
`createVersion` accepts arbitrary client S3 keys

**Status: Accepted** — team decision; Joi length validation only.

Risk Level: HIGH  
File Path: src/services/edit-request-output.service.ts  
Lines: 447-484

Description:
`createVersion` persists `body.key` and `thumbnailKey` from the client without server-issued key binding.

Impact:
- Client could register arbitrary keys if IDs are known (accepted product risk)

Recommendation:
None for SKSH-116.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Unescaped `$regex` in admin list search | HIGH | ✅ Fixed | `src/services/edit-request.service.ts` | 200, 227 |
| 2 | No ownership check on replace/remove source video | HIGH | Accepted | `src/services/edit-request.service.ts` | — |
| 3 | `/file` stream route skips download audit | HIGH | ✅ Fixed | `src/routes/edit-request.routes.ts` | — |
| 4 | Admin `submitFeedback` sets `approved` | HIGH | ✅ Fixed | `src/services/edit-request.service.ts` | — |
| 5 | N+1 in output/version mapping | HIGH | ✅ Fixed | `src/services/edit-request-output.service.ts` | 326-361 |
| 6 | Socket join loads all admin user IDs | HIGH | Accepted | `src/services/edit-request.service.ts` | — |
| 7 | Unbounded user `$in` for admin search | HIGH | ✅ Fixed | `src/repositories/user.repository.ts` | — |
| 8 | `createVersion` arbitrary S3 keys | HIGH | Accepted | `src/services/edit-request-output.service.ts` | 447-484 |

**Positive notes:** Re-reviewed @ `f65ed2b`. Commits since `66514da` are main/SKSH-295 merges only — **no changes** to `edit-request*` services, controllers, or routes. Original **5 fixed / 3 accepted** still verified (`containsInsensitive`, batch output mapping, etc.). **0 new** Critical/High.

**Skipped:** Pagination-related items per prior review request. Medium/Low unless asked.
