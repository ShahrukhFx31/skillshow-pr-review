# Frontend PR Review — skillshow-admin-ui (`SKSH-295`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-295` @ `8fa4fc98` / `f0d3be22`  
**Re-verified:** May 2026 (no new commits on `origin/SKSH-295` since `f0d3be22`)  
**Scope:** Crew onboarding wizard, crew dashboard, routing/auth redirect, shared state/city selector (Critical, High, Medium)  
**Findings:** 4 original + 1 optional Medium — **3 fixed**, **1 accepted**, **0 open**, **0 new Critical/High**

---

---
Weekday availability buttons use duplicate labels

Risk Level: HIGH  
File Path: src/pages/dashboard/crew/constants.ts  
Lines: 58-66

Description:
`CREW_WEEKDAYS` assigned label `"T"` to both Tuesday (`code: "T"`) and Thursday (`code: "R"`), and `"S"` to both Saturday (`code: "S"`) and Sunday (`code: "U"`). `WeekdayToggleGroup` renders these labels on circle buttons, so users could not distinguish Thu vs Tue or Sun vs Sat when selecting availability.

Impact:
- Crew may save wrong `typicalDaysAvailable` codes (e.g. Tuesday selected when they meant Thursday)
- Downstream scheduling/assignment uses incorrect availability

Recommendation:
Use unique labels per code, e.g. Thu → `"R"`, Sun → `"U"`.

**PR comment (line 62):** **High:** Thursday and Sunday toggles show the same labels as Tuesday/Saturday (`T`/`S`), so availability selection is ambiguous—please use distinct labels per weekday code.

**Re-review:** ✅ Fixed @ `f0d3be22` — Still correct: `R` / `U` labels at lines 62–65.

---

---
`CheckableTags` reads form values without subscribing to updates

Risk Level: MEDIUM  
File Path: src/pages/dashboard/crew/onboarding/steps/job-preferences-step.tsx  
Lines: 23-24, 117-120

Description:
`CheckableTags` for `otherRoles` used `form.getFieldValue("otherRoles")` instead of `Form.useWatch`, so tag UI could desync after hydrate (same pattern existed in `filming-editing-step.tsx` for `editingSpecialties`).

Impact:
- Stale UI: selected tags appear unselected until another field triggers a re-render
- Users may resubmit or misread their selections

Recommendation:
Use `Form.useWatch` for `otherRoles`, `typicalDaysAvailable`, and `editingSpecialties`.

**PR comment (line 120):** **Medium:** `otherRoles` uses `getFieldValue` so tag selection can desync after hydrate—use `Form.useWatch` for controlled updates.

**Re-review:** ✅ Fixed @ `f0d3be22` — `useWatch` on `otherRoles`, `typicalDaysAvailable`, `editingSpecialties`.

---

---
Post-verify redirect ignores crew onboarding completion

**Status: Accepted** — team decision; low-frequency path (re-verify after onboarding). Login and `ProtectedRoute` already gate crew via `needsCrewOnboarding`; follow-up optional.

Risk Level: MEDIUM  
File Path: src/utils/auth-redirect.ts  
Lines: 27, 38-40, 72-75 (72–75 unchanged in PR — not commentable on GitHub diff)

Description:
`getPostVerifyRouteForUser` (lines 72–75) resolves `roleKey` and returns `ROLE_POST_VERIFY_ROUTES[roleKey]` without checking `isCrewOnboarded`, unlike `getDefaultRouteForUser` (lines 54–55), which uses `needsCrewOnboarding`. This PR adds `crew` to `ROLE_ONBOARDING_ROUTES` (line 27); that entry overrides `crew` in `ROLE_POST_VERIFY_ROUTES` (lines 38–40). Onboarded crew who re-verify email are still sent to `/crew-onboarding`.

Impact:
- Confusing redirect after email verification for onboarded crew
- Extra navigation step; risk of accidental “Skip”/complete calls

Recommendation:
At the top of `getPostVerifyRouteForUser`, mirror login:

```ts
if (needsCrewOnboarding(userInfo)) {
  return ROLE_ONBOARDING_ROUTES.crew;
}
```

**GitHub PR comment (line 54):** See prior review — comment on the `needsCrewOnboarding` line in `getDefaultRouteForUser` and ask to mirror in `getPostVerifyRouteForUser`.

**Re-review:** Accepted — not implemented (by design). Re-checked May 2026; `getPostVerifyRouteForUser` unchanged.

---

---
`OnboardingSection` uses ternary-with-null for optional description

Risk Level: MEDIUM  
File Path: src/pages/dashboard/crew/onboarding/ui.tsx  
Lines: 22

Description:
Optional description used `description ? … : null` instead of `description && …`.

Impact:
- Minor consistency/maintainability only; no functional bug

Recommendation:
Replace with `description && <Typography.Text type="secondary">{description}</Typography.Text>`.

**PR comment (line 22):** **Medium (style):** Prefer `description && <Typography.Text>…</Typography.Text>` over ternary `: null` for optional fragments.

**Re-review:** ✅ Fixed @ `f0d3be22` — Uses `description && …`.

---

---
(Optional) `SegmentedToggle` for computer type uses `getFieldValue`

Risk Level: MEDIUM  
File Path: src/pages/dashboard/crew/onboarding/steps/equipment-step.tsx  
Lines: 47-50

Description:
`computerType` `SegmentedToggle` uses `value={form.getFieldValue("computerType")}` instead of `Form.useWatch("computerType", form)`, same class of issue as the fixed `CheckableTags` findings.

Impact:
- Computer type toggle may not reflect hydrated profile until another field re-renders

Recommendation:
`const computerType = Form.useWatch("computerType", form);` and pass to `SegmentedToggle`.

**Re-review:** Open (optional) — not in original review; low severity; fix in a follow-up if desired. **Not blocking merge.**

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Weekday toggle duplicate labels (T/S) | HIGH | ✅ Fixed | `src/pages/dashboard/crew/constants.ts` | 58-66 |
| 2 | `CheckableTags` uses `getFieldValue` not `useWatch` | MEDIUM | ✅ Fixed | `src/pages/dashboard/crew/onboarding/steps/job-preferences-step.tsx` | 23-24, 117-120 |
| 3 | Post-verify redirect ignores `isCrewOnboarded` | MEDIUM | Accepted | `src/utils/auth-redirect.ts` | 27, 54-55, 72-75 |
| 4 | Ternary-with-null for optional description | MEDIUM | ✅ Fixed | `src/pages/dashboard/crew/onboarding/ui.tsx` | 22 |
| 5 | `computerType` toggle uses `getFieldValue` (optional) | MEDIUM | Open (optional) | `src/pages/dashboard/crew/onboarding/steps/equipment-step.tsx` | 47-50 |

**Branch status:** No new commits on `origin/SKSH-295` after `f0d3be22`. Latest fixes are present on the checked branch.

**Positive notes:** Form controls largely reactive; weekday labels correct; crew gating via `ProtectedRoute` + login redirect intact.

**New issues:** One optional Medium (#5), same pattern as fixed #2 — not Critical/High.

**Skipped (per prompt):** Low/Info; ad-hoc grays and Ant `!` overrides; palette vs design tokens.
