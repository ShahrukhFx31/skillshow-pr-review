# Backend PR Review — skillshow (`SKSH-329`)

**Repo:** skillshow — `https://github.com/fx31labs-mvp/skillshow.git`  
**Branch:** `SKSH-329`  
**Base:** `main...HEAD` @ `5c3b6c6`  
**Initial review:** 2026-06-12  
**Re-reviewed:** 2026-06-12 (`5c3b6c6` — manager history defaults added)  
**Scope:** Rename user-facing “internal revision” default strings to “internal review” in output controller + edit-request service (Critical / High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [frontend.md](./frontend.md)

**Findings:** 1 (0 Critical, 1 High) — **0 Open**, **1 Fixed**

### Protected modules

| Module | Status |
|--------|--------|
| `list-query.validation.ts`, `list-query-aggregation.utils.ts` | Not modified |
| Audit-log modules, change-stream modules | Not modified |

### Files reviewed (2 changed)

| File | Change |
|------|--------|
| `src/controllers/edit-request-output.controller.ts` | OK/error message: “Sent for internal review” |
| `src/services/edit-request.service.ts` | Crew default note + manager history defaults → “internal review” |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Minimal string-only diff — no new abstractions | ✅ KISS |
| API response messages match new frontend toast/labels | ✅ |
| Persisted manager approve/changes history defaults | ✅ Fixed (#1) |
| `edit-request-output.constants.ts` upload-hint strings unchanged (“internal revision”) | ⚠️ Out of PR diff; code comments / crew upload hints — follow-up optional |
| Protected list/audit modules untouched | ✅ |

### Positive notes

- **Complete in-scope migration:** Crew send-to-manager, controller messages, and manager approve/changes history defaults all use “internal review”.
- **Cross-stack:** Aligns with frontend activity-history labels and `appToast.success("Sent for internal review.")`.

---

## GitHub comments

_No open findings — prior comment resolved on branch._

---

## Findings

---
Persisted history notes still use “internal revision” wording

Risk Level: HIGH  
**Status:** ✅ Fixed  
File Path: skillshow/src/services/edit-request.service.ts  
Lines: 788-797

Description:
**Global consistency.** Initial review flagged `reviewInternalRevision` persisting `"Manager approved internal revision"` and `"Manager requested changes on internal revision"`.

Impact:
- Mixed terminology on manager approve/changes actions.

Recommendation:
Update persisted defaults to “internal review”.

**Re-review evidence:** Branch now writes:

```typescript
note: "Manager approved internal review",
note: body.note?.trim() || "Manager requested changes on internal review",
```

---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Persisted history notes still use “internal revision” wording | HIGH | ✅ Fixed | skillshow/src/services/edit-request.service.ts | 788-797 | — |

**Merge readiness:** **Merge-ready** — no open Critical/High blockers. Frontend ([#3](./frontend.md), [#7](./frontend.md)) still has open High items.
