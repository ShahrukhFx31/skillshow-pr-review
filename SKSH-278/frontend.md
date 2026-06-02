# SKSH-278 — Frontend review (`skillshow-admin-ui`)

**Branch:** `sksh-278`  
**Base:** `main`  
**Re-review:** `849b3da8` (`fix: feedback`)  
**Scope:** Profile photo delete UX; team logo upload preview/delete overlay; stale avatar sync fix; `suppressSuccessToast` for silent team PATCH; shared `UploadedImageOverlay` component.

## Overview

Adds delete/view overlay for profile photos and team logos, wires `deleteProfileImage` API, fixes header avatar sticking to an old uploaded URL after removal (`isUploadedProfileImageUrl` + store sync), and allows clearing team logos via backend `logoKey: ""`.

**Re-review (`849b3da8`):** All five Medium findings are verified fixed. No new Critical, High, or Medium issues introduced.

---

## Findings

No **Critical**, **High**, or **Medium** findings.

---

## Prior findings (verified)

### 1. Non-atomic saved team logo removal — ✅ Fixed

**Evidence:** Saved-logo path PATCHes `logoKey: ""` first, then `deleteProfileImage`; S3 failure shows warning; PATCH failure skips S3 and invalidates queries.

### 2. Cancel restores stale logo after persisted delete — ✅ Fixed

**Evidence:** `logoDeletePersisted`; conditional `cancelEdit` logic; `useEffect` clears `logoDeletePersisted` when team has no logo.

### 3. Logo delete PATCH bypasses form validation — ✅ Fixed

**Evidence:** `await form.validateFields()` before `updateCoachTeam` on saved-logo delete.

### 4. `beginEdit` resets logo guards before refetched `team` props — ✅ Fixed

**Evidence (`1dc82b81`):** `setLogoRemoved(logoDeletePersisted || teamHasNoLogo)` in `beginEdit`.

### 5. Cancel after persisted delete + unsaved re-upload shows stale logo — ✅ Fixed

**Evidence (`849b3da8`):**

```76:80:src/pages/teams/details/components/TeamOverviewSection.tsx
		if (logoDeletePersisted) {
			setLogoRemoved(true);
		} else {
			setLogoRemoved(false);
		}
```

Also clears persisted-delete flags on successful save when a new `pendingLogoKey` is saved (`onSave` lines 117–120).

---

## Positive notes

- **`UploadedImageOverlay`:** Reusable view/delete overlay with `cn()`, a11y labels, and touch-friendly always-visible actions on non-hover devices.
- **Profile avatar sync:** `isUploadedProfileImageUrl` + revised `useEffect` stops the header from keeping a removed S3 URL; falls back to ui-avatars placeholder correctly.
- **`suppressSuccessToast`:** Avoids double toasts when logo delete chains a silent team PATCH.
- **Logo state machine:** PATCH-first delete, `logoDeletePersisted`, `beginEdit` / `cancelEdit` / `onSave` flags form a consistent stale-prop strategy across delete, cancel, re-edit, and save paths.
- **Create vs edit team logo UX:** Preview lives outside `Upload.Dragger` when set, so delete/view actions do not fight the drop zone.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Non-atomic saved team logo removal | MEDIUM | ✅ Fixed | src/pages/teams/details/components/TeamOverviewSection.tsx | 178-201 |
| 2 | Cancel restores stale logo after persisted delete | MEDIUM | ✅ Fixed | src/pages/teams/details/components/TeamOverviewSection.tsx | 36, 55-59, 73-86 |
| 3 | Logo delete PATCH bypasses form validation | MEDIUM | ✅ Fixed | src/pages/teams/details/components/TeamOverviewSection.tsx | 179 |
| 4 | `beginEdit` resets logo guards before refetched `team` props | MEDIUM | ✅ Fixed | src/pages/teams/details/components/TeamOverviewSection.tsx | 68-69 |
| 5 | Cancel after persisted delete + unsaved re-upload shows stale logo | MEDIUM | ✅ Fixed | src/pages/teams/details/components/TeamOverviewSection.tsx | 76-80, 117-120 |

*No open Critical/High/Medium findings.*
