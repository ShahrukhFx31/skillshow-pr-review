# SKSH-278 — Frontend review (`skillshow-admin-ui`)

**Branch:** `sksh-278`  
**Base:** `main`  
**Re-review:** `f76b807b` (HEAD; includes `849b3da8` feedback fixes + merge from `main`)  
**Scope:** Profile photo delete UX; team logo upload preview/delete overlay; stale avatar sync; `suppressSuccessToast`; `UploadedImageOverlay`.

## Overview

Adds delete/view overlay for profile and team images, `deleteProfileImage` API wiring, header avatar sync after removal (`isUploadedProfileImageUrl`), and team `logoKey: ""` clearing via coach PATCH. Team logo edit flow uses PATCH-first delete, `logoDeletePersisted`, and coordinated `beginEdit` / `cancelEdit` / `onSave` state.

**Final re-review:** All five prior Medium findings re-verified on current HEAD. No new Critical, High, or Medium issues in PR scope.

---

## Findings

No **Critical**, **High**, or **Medium** findings.

---

## Prior findings (verified on HEAD)

| # | Finding | Status | Evidence |
|---|---------|--------|----------|
| 1 | Non-atomic saved team logo removal | ✅ Fixed | PATCH `logoKey: ""` then S3 delete; warning on S3-only failure (`178-201`) |
| 2 | Cancel restores stale logo after persisted delete | ✅ Fixed | `logoDeletePersisted` + conditional `cancelEdit` (`76-80`) |
| 3 | Logo delete PATCH bypasses form validation | ✅ Fixed | `await form.validateFields()` (`179`) |
| 4 | `beginEdit` resets logo guards before refetch | ✅ Fixed | `setLogoRemoved(logoDeletePersisted \|\| teamHasNoLogo)` (`68-69`) |
| 5 | Cancel after delete + unsaved re-upload | ✅ Fixed | `cancelEdit` sets `logoRemoved(true)` when persisted (`76-80`); `onSave` clears flags on new logo save (`117-120`) |

---

## Positive notes

- **`UploadedImageOverlay`:** Reusable component in `src/components/upload/` with `cn()`, a11y labels, preview via Ant `Image`.
- **Profile flow:** Delete refetches account general, updates query cache and header store; `hasUploadedProfilePhoto` gates delete overlay.
- **Team logo state machine:** Handles orphan S3 keys, persisted delete, cancel, re-edit, and save without stale `team.logoUrl` flashes in normal paths.
- **`suppressSuccessToast`:** Prevents duplicate toasts on silent team PATCH during logo removal.
- **Unrelated diff hunks** (pagination bar, video hooks, antd adapter) are formatting-only from merge.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Non-atomic saved team logo removal | MEDIUM | ✅ Fixed | src/pages/teams/details/components/TeamOverviewSection.tsx | 178-201 |
| 2 | Cancel restores stale logo after persisted delete | MEDIUM | ✅ Fixed | src/pages/teams/details/components/TeamOverviewSection.tsx | 36, 55-59, 73-86 |
| 3 | Logo delete PATCH bypasses form validation | MEDIUM | ✅ Fixed | src/pages/teams/details/components/TeamOverviewSection.tsx | 179 |
| 4 | `beginEdit` resets logo guards before refetched `team` props | MEDIUM | ✅ Fixed | src/pages/teams/details/components/TeamOverviewSection.tsx | 68-69 |
| 5 | Cancel after persisted delete + unsaved re-upload shows stale logo | MEDIUM | ✅ Fixed | src/pages/teams/details/components/TeamOverviewSection.tsx | 76-80, 117-120 |

**Merge readiness:** No open Critical/High/Medium blockers.
