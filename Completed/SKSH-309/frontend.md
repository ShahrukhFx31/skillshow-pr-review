# Frontend PR Review — skillshow-admin-ui (`SKSH-309`)

**Repo:** skillshow-admin-ui — `git@github-work:SkillshowFx/skillshow-admin-ui.git`  
**Branch:** `SKSH-309`  
**Base:** `main...HEAD` @ `5ea29495`  
**Initial review:** 2026-06-19  
**Scope:** US phone input components with caret-safe formatting, edit-request icon/copy centralization, edit-request service tiles refactor (Critical / High / Medium only)  
**Prompts:** `frontend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Findings:** 1 (0 Critical, 1 High) — **0 Open** (1 Accepted)

### Protected modules

| Module | Status |
|--------|--------|
| `pagination-bar.tsx`, `use-server-table-controls.ts`, `audit-log/*`, `table-empty-state.tsx`, `antd.adapter.tsx` | **Not modified** ✅ |

### Files reviewed

| File | Change |
|------|--------|
| `src/components/forms/UsPhoneInput.tsx` | New caret-safe US phone input (`antd` / `native` variants) |
| `src/components/forms/UsPhoneFormField.tsx` | New `+1`-prefixed Ant Design form field wrapper |
| `src/components/forms/index.ts` | Export new phone components |
| `src/ui/input.tsx` | `forwardRef` for native phone input integration |
| `src/constants/common.constant.ts` | `EDIT_REQUEST_ICON` |
| `src/pages/auth/components/auth-ui.tsx` | Export `INPUT_BASE_CLASSES` |
| `src/pages/auth/components/register/PhoneField.tsx` | Migrate to `UsPhoneInput` (native) |
| `src/pages/management/*/onboarding/components/*-form.tsx` | Replace inline `Space.Compact` phone blocks with `UsPhoneFormField` |
| `src/pages/user/account/general/components/ContactInformationCard.tsx` | Migrate to `UsPhoneInput` (native) |
| `src/pages/dashboard/crew/shared/sections/background-tax/BackgroundTaxEditFields.tsx` | Migrate to `UsPhoneInput` |
| `src/pages/dashboard/constant/index.tsx` | `EDIT_REQUEST_ICON` |
| `src/pages/dashboard/parent/index.tsx` | `EDIT_REQUEST_ICON` |
| `src/pages/dashboard/superAdmin/constants.ts` | `EDIT_REQUEST_ICON` |
| `src/pages/dashboard/crew/dashboard/components/edit-requests-columns.tsx` | Goal tag `w-fit` |
| `src/pages/editRequest/constants/editRequest.constants.ts` | `EDIT_REQUEST_INCLUDED_SERVICES`, pricing/title constants |
| `src/pages/editRequest/components/SkillServiceTiles.tsx` | Consume shared service list |
| `src/pages/editRequest/components/UploadVideosScreen.tsx` | Shared title/pricing copy |
| `src/pages/editRequest/components/EnhanceVideoScreen.tsx` | Shared service bullets/pricing prefix |
| `src/pages/editRequest/components/EditRequestListToolbar.tsx` | `EDIT_REQUEST_ICON`, control sizing |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| `UsPhoneInput` centralizes `reconcileUsPhoneField` + caret preservation | ✅ DRY |
| `UsPhoneFormField` removes duplicated `Space.Compact` `+1` blocks across 4 onboarding forms | ✅ DRY / Reusability |
| `EDIT_REQUEST_ICON` migrated across dashboard + edit-request toolbar surfaces touched in PR | ✅ Global consistency |
| `EDIT_REQUEST_INCLUDED_SERVICES` shared by `SkillServiceTiles` and `EnhanceVideoScreen` bullets | ✅ DRY |
| `normalize` on phone field specs in crew/partner/skillshow constants no longer wired in forms | ✅ Accepted — dead config after `UsPhoneInput` migration; harmless, optional cleanup |
| `verify-email.constants.ts` still uses legacy `fluent:image-edit-16-filled` | ✅ Accepted — not in PR diff |
| EnhanceVideoScreen welcome banner reuses service-card title constant | ✅ Accepted — intentional copy; see finding #1 |

### Positive notes

- Caret-safe formatting via `caretAfterFormat` + `useLayoutEffect` is a solid UX improvement over plain `normalize` on Ant `Form.Item`.
- `UsPhoneInput` correctly supports both Ant Design Form (`onChange`) and react-hook-form (`onValueChange`) call sites.
- `ui/input.tsx` `forwardRef` change is minimal and required for RHF `field.ref` wiring.
- Edit-request service tiles now show all six included services from one constant source.

---

## GitHub comments

### `src/pages/editRequest/components/EnhanceVideoScreen.tsx` (L72–74)

When `EDIT_REQUEST_ENHANCE_BANNER_TITLE` was removed, the welcome banner was wired to the local `EDIT_REQUEST_ENHANCE_SERVICE_TITLE` (`"Service Offered By SkillShow"`), which is the service-card heading — not the prior banner copy (`"Enhance your Video"`). The shared constant added in `editRequest.constants.ts` (`"Enhance Your Video"`) is used on `UploadVideosScreen` but not here, so the two edit-request steps now show mismatched primary headings. Please restore a dedicated banner title (import the shared constant or add `EDIT_REQUEST_ENHANCE_BANNER_TITLE`) and keep the service-card title separate.

---

## Findings

---
EnhanceVideoScreen welcome banner shows service-card title instead of banner copy

Risk Level: HIGH
File Path: src/pages/editRequest/components/EnhanceVideoScreen.tsx
Lines: 36, 72-74

Description:
**Global consistency** / UX regression. On `main`, `GradientWelcomeBanner` used `EDIT_REQUEST_ENHANCE_BANNER_TITLE` (`"Enhance your Video"`), while the right-hand service card used a separate local `EDIT_REQUEST_ENHANCE_SERVICE_TITLE` (`"Service Offered By SkillShow"`). This PR removed `EDIT_REQUEST_ENHANCE_BANNER_TITLE` and passed the service-card title into the banner (`title={EDIT_REQUEST_ENHANCE_SERVICE_TITLE}`). It also introduced a shared `EDIT_REQUEST_ENHANCE_SERVICE_TITLE = "Enhance Your Video"` in `editRequest.constants.ts` (consumed by `UploadVideosScreen`) but `EnhanceVideoScreen` keeps a **shadowing local constant** with the same name and a different value. Users now see `"Service Offered By SkillShow"` on the enhance step banner instead of the prior enhance-focused headline, while the upload step shows `"Enhance Your Video"`.

Impact:
- Inconsistent primary headings across the edit-request create flow (upload vs enhance steps).
- Welcome banner copy no longer matches the intended “enhance your video” messaging from `main`.
- Duplicate constant name with divergent values makes future copy updates error-prone.

Recommendation:
Add a dedicated banner constant and use the shared title only where intended:

```ts
// editRequest.constants.ts
export const EDIT_REQUEST_ENHANCE_BANNER_TITLE = "Enhance Your Video";
export const EDIT_REQUEST_ENHANCE_SERVICE_TITLE = "Service Offered By SkillShow";

// EnhanceVideoScreen.tsx
import {
  EDIT_REQUEST_ENHANCE_BANNER_TITLE,
  EDIT_REQUEST_ENHANCE_SERVICE_TITLE,
  // ...
} from "../constants/editRequest.constants";

<GradientWelcomeBanner
  description={EDIT_REQUEST_ENHANCE_BANNER_DESCRIPTION}
  title={EDIT_REQUEST_ENHANCE_BANNER_TITLE}
/>
```

Keep `EDIT_REQUEST_ENHANCE_SERVICE_TITLE` on the service card section only (L229). Remove the shadowing local const at L36.

**PR comment:** See GitHub comments above.

**Accepted (2026-06-19):** Reviewer finding acknowledged; banner title reuse is intentional — no change required before merge.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | EnhanceVideoScreen welcome banner shows service-card title instead of banner copy | HIGH | Accepted | src/pages/editRequest/components/EnhanceVideoScreen.tsx | 36, 72-74 |

**Merge readiness:** ✅ No open Critical/High/Medium blockers. Finding #1 accepted as intentional copy.
