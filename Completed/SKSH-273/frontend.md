# Frontend PR Review — skillshow-admin-ui (`SKSH-273`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-273`  
**Base:** `main...HEAD`  
**Scope:** React performance, hooks, JSX/props, Tailwind/file structure (Critical, High, Medium only)  
**Findings:** 3 (0 Critical, 2 High, 1 Medium) — **all resolved**

---

---
Dead fallback calls non-existent profile API

Risk Level: HIGH
File Path: src/pages/share/video/index.tsx

Description:
Query previously called `GET /public/videos/:id/profile` when embedded profile was null; route did not exist.

Impact:
- Extra 404 on share page loads

Recommendation:
Remove fallback; use embedded profile from main endpoint.

**Re-verification:** ✅ Fixed — `queryFn: () => getPublicVideoShare(videoId)` only; `getPublicVideoOwnerProfile` removed.

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

**Re-verification:** ✅ Fixed — backend requires explicit `true`; `useVideoUpload` hydrates `isPublic` and `shareUrl` from create response.

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
Use `&&` for optional fragments.

**Re-verification:** ✅ Fixed — `subtitle && <p>…</p>` and `showProfile && profile && (<div>…</div>)` (no null fallback).

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Dead fallback calls non-existent `/profile` API | HIGH | ✅ Fixed | `src/pages/share/video/index.tsx` | — |
| 2 | Upload public switch vs backend `isPublic` semantics | HIGH | ✅ Fixed | `src/pages/videos/utils/video-public.utils.ts` | 1-4 |
| 3 | Ternary-with-null for optional JSX | MEDIUM | ✅ Fixed | `src/pages/share/video/components/PublicShareMediaLayout.tsx` | 47, 51-55 |

**Positive notes:** Public route at `/share/video/:id`; `apiClient` 401 handling for public URLs; role-scoped share profile display; feature colocated under `src/pages/share/video/`.

**Merge readiness:** No open Critical/High/Medium blockers.
