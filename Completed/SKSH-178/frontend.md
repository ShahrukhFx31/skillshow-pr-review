# SKSH-178 — Frontend review (`skillshow-admin-ui`)

**Repo:** skillshow-admin-ui  
**Branch:** `SKSH-178-1`  
**Base:** `main...HEAD`  
**Latest re-review:** `0555c541` (HEAD; no newer commits on `origin/SKSH-178-1`)  
**Scope:** App users management + related video UI (Critical / High / Medium only)

**Review note:** Pagination-related items are **out of scope**.

---

## Re-review summary (latest pass)

| # | Finding | Verdict |
|---|---------|---------|
| 1 | Onboarding `constants.ts` monolith | **✅ Fixed** (`0555c541`) |
| 2 | `VideoExpandedPlatformsRow` throwing pending hook | **✅ Fixed** (`0555c541`) |

**New commits since `0555c541`:** none.

**New Critical / High / Medium findings:** none. **Archived:** `pr-review/Completed/SKSH-178/`.

---

## Overview

No open in-scope findings on current `SKSH-178-1` HEAD. Activity module, onboarding split, detail route guard, and optional video pending hook remain in good shape.

---

## Positive notes

- **`activity/`** module with columns, table, filters.
- **`onboarding/utils.ts`** for validation + API payloads; slimmer `constants.ts`.
- **`appUserDetailMatchesRoute`** + `activeDetail` prevents stale detail flash.
- **`useVideoListPendingOptional`** in `VideoExpandedPlatformsRow.tsx` (line 18).

---

## Findings

---
Onboarding `constants.ts` mixes form config, validation rules, and API payloads

Risk Level: MEDIUM  

**Re-review:** **✅ Fixed** in `0555c541`.

---

---
Desktop expanded platform row still uses throwing pending hook

Risk Level: MEDIUM  

**Re-review:** **✅ Fixed** in `0555c541` — `useVideoListPendingOptional`.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Onboarding `constants.ts` mixes form config, validation rules, and API payloads | MEDIUM | ✅ Fixed | src/pages/management/app-users/onboarding/constants.ts | 1-202 |
| 2 | Desktop expanded platform row still uses throwing pending hook | MEDIUM | ✅ Fixed | src/pages/videos/components/VideoExpandedPlatformsRow.tsx | 17-18 |

*No open in-scope findings.*

### Out of scope (pagination — not reported)

| Title | Status |
|--------|--------|
| Activity summary cards vs API video totals | Accepted — deferred |
| Client-side activity video paging / full payload in memory | Accepted — deferred |
| Bulk update UI vs 50-user API cap | Accepted — deferred; API validates on submit |

---

## Merge readiness

**Complete** — all in-scope findings are **Fixed** or **Accepted**; no **Open** rows. Archived to `pr-review/Completed/SKSH-178/`.
