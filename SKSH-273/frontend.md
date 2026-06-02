# Frontend PR Review — skillshow-admin-ui (`SKSH-273`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-273`  
**Base:** `main...HEAD`  
**Scope:** React performance, hooks, JSX/props, Tailwind/file structure (Critical, High, Medium only)  
**Findings:** 3 (0 Critical, 2 High, 1 Medium)

---

---
Dead fallback calls non-existent profile API

Risk Level: HIGH
File Path: src/pages/share/video/index.tsx
Lines: 27-35

Description:
After `getPublicVideoShare`, when `share.profile` is null the query calls `getPublicVideoOwnerProfile(videoId)`, which requests `GET /public/videos/:id/profile`. The backend PR only implements `GET /public/videos/:id`—there is no `/profile` route. Every share page load with a missing embedded profile therefore issues an extra failing request (404) before settling.

Impact:
- Wasted network round-trip and error noise on every view where profile is omitted
- Harder to debug real API failures; obscures missing backend profile data

Recommendation:
Remove the fallback until a backend route exists, or drop `getPublicVideoOwnerProfile` from `publicVideoService.ts` entirely. The main payload already includes `profile` from `VideoPublicService.getPublicShareView`.

**PR comment (line 30):**  
**High:** This fallback hits `/public/videos/:id/profile`, which is not implemented on the API—please remove it or add the matching backend route so we do not 404 on every share load without an embedded profile.

---

---
Upload “Public” switch off does not match backend share eligibility

Risk Level: HIGH
File Path: src/pages/videos/utils/video-public.utils.ts
Lines: 1-4

Description:
`isVideoPublic` returns true only when `isPublic === true`. The upload `Switch` uses this helper and hides `shareUrl` when false. The backend still treats `isPublic !== false` as shareable and defaults new videos to `isPublic: true`. The PR also changed `VideoItem` from `checked={videoLevelData.isPublic ?? true}` to strict `isVideoPublic(...)`, so the switch defaults off while many rows remain publicly shareable server-side until an explicit `PATCH` sets `isPublic: false`.

Impact:
- Users see “private” in the upload UI but may still have a working public link (from list `shareUrl` or guessed URL)
- Mismatch between copy-link visibility and actual access

Recommendation:
Coordinate with backend to require `isPublic === true` for share (see backend review). On the client, after upload completes, hydrate `isPublic` / `shareUrl` from the created video response so UI state matches the server before the user toggles.

**PR comment (line 2):**  
**High:** UI treats only `isPublic === true` as public, but the API still shares videos when `isPublic` is undefined/default true. Please align backend rules and sync initial upload state from the API so the switch reflects real shareability.

---

---
JSX uses ternary-with-null for optional fragments

Risk Level: MEDIUM
File Path: src/pages/share/video/components/PublicShareMediaLayout.tsx
Lines: 47, 51-55

Description:
Optional UI uses `subtitle ? <p>...</p> : null` and `showProfile && profile ? <div>...</div> : null` instead of logical AND when the false branch renders nothing.

Impact:
- Inconsistent with project JSX conventions; slightly noisier diffs

Recommendation:
Use `subtitle && <p className="...">...</p>` and `showProfile && profile && <div>...</div>` (drop redundant ternary fallbacks).

**PR comment (line 47):**  
**Medium:** Prefer `subtitle && <p>…</p>` over `subtitle ? <p>…</p> : null` per our JSX conditional rendering guideline.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Dead fallback calls non-existent `/profile` API | HIGH | Open | `src/pages/share/video/index.tsx` | 27-35 |
| 2 | Upload public switch vs backend `isPublic` semantics | HIGH | Open | `src/pages/videos/utils/video-public.utils.ts` | 1-4 |
| 3 | Ternary-with-null for optional JSX | MEDIUM | Open | `src/pages/share/video/components/PublicShareMediaLayout.tsx` | 47, 51-55 |

**Positive notes:** Public route registered without auth at `/share/video/:id`. `apiClient` correctly skips session clear/redirect on 401 for `PUBLIC_VIDEOS_PREFIX`. Feature colocated under `src/pages/share/video/` with components, constants, and utils. `ShareUrlCopyField` reused across table, mobile card, and upload panel. `ResizeObserver` in `PublicShareMediaLayout` cleans up on unmount. `cn()` used for layout classes.

**Skipped (per prompt):** Ad-hoc slate/blue palette and Ant `!` overrides (out of scope). Low-severity items (empty `<track>`, `SportProfileItem` typing on public profile, duplicate `SHARE_PAGE_EDIT_TIP` string).

**Re-review update (latest `SKSH-273`):** All 3 findings re-verified and still reproducible; no new frontend Critical/High/Medium issues were introduced by the latest role/display updates.

**Merge readiness:** Blocked — 2 open High and 1 open Medium findings.
