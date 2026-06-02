# Frontend PR Review — skillshow-admin-ui (`SKSH-273`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-273`  
**Base:** `main...HEAD`  
**Scope:** React performance, hooks, JSX/props, Tailwind/file structure (Critical, High, Medium only)  
**Findings:** 3 (0 Critical, 2 High, 1 Medium) — **re-verified: 2 ✅ Fixed, 1 Partially fixed**

---

---
Dead fallback calls non-existent profile API

Risk Level: HIGH
File Path: src/pages/share/video/index.tsx
Lines: 27-35 (original)

Description:
After `getPublicVideoShare`, when `share.profile` was null the query called `GET /public/videos/:id/profile`, which was not implemented on the backend.

Impact:
- Extra 404 on every share load without embedded profile

Recommendation:
Remove fallback; rely on embedded `profile` from main endpoint.

**Re-verification (commits `4ffd3a00` / `721b0842`):** ✅ Fixed — `queryFn` is `() => getPublicVideoShare(videoId)` only; `getPublicVideoOwnerProfile` removed from `publicVideoService.ts`.

---

---
Upload “Public” switch off does not match backend share eligibility

Risk Level: HIGH
File Path: src/pages/videos/utils/video-public.utils.ts
Lines: 1-4

Description:
UI used `isPublic === true` while backend treated `undefined` as public.

Impact:
- Mismatch between switch/copy-link and actual share access

Recommendation:
Align backend; hydrate upload state from API after create.

**Re-verification:** ✅ Fixed — backend now requires explicit `isPublic === true`; `useVideoUpload` hydrates `isPublic` and `shareUrl` from `createdVideo` response.

---

---
JSX uses ternary-with-null for optional fragments

Risk Level: MEDIUM
File Path: src/pages/share/video/components/PublicShareMediaLayout.tsx
Lines: 47, 51-55

Description:
Optional UI used `condition ? <Node /> : null` instead of logical AND.

Impact:
- Style/convention inconsistency

Recommendation:
Use `subtitle && <p>…</p>` and `showProfile && profile && <div>…</div>`.

**Re-verification:** Partially fixed — subtitle uses `&&` (line 47); profile sidebar still uses `showProfile && profile ? (…) : null` (lines 51-55). Optional one-line cleanup remains.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Dead fallback calls non-existent `/profile` API | HIGH | ✅ Fixed | `src/pages/share/video/index.tsx` | — |
| 2 | Upload public switch vs backend `isPublic` semantics | HIGH | ✅ Fixed | `src/pages/videos/utils/video-public.utils.ts` | 1-4 |
| 3 | Ternary-with-null for optional JSX | MEDIUM | Partially fixed | `src/pages/share/video/components/PublicShareMediaLayout.tsx` | 51-55 |

**Positive notes:** Public route at `/share/video/:id`; `apiClient` 401 handling for public URLs; role-scoped share profile display; feature colocated under `src/pages/share/video/`.

**Merge readiness:** No open Critical/High blockers. One optional Medium style cleanup (profile block ternary) — acceptable to merge or fix in follow-up.
