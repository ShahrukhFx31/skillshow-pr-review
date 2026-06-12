# Backend PR Review — skillshow (`SKSH-329`)

**Repo:** skillshow — `https://github.com/fx31labs-mvp/skillshow.git`  
**Branch:** `SKSH-329`  
**Base:** `main...HEAD` @ `f2ff297`  
**Initial review:** 2026-06-12  
**Scope:** Rename user-facing “internal revision” default strings to “internal review” in output controller + edit-request service (Critical / High only)  
**Prompts:** `backend-system-prompt.md` (DRY / KISS / Global consistency / Contract / protected modules)

**Aligned with:** [frontend.md](./frontend.md)

**Findings:** 1 (0 Critical, 1 High) — **1 High Open**

### Protected modules

| Module | Status |
|--------|--------|
| `list-query.validation.ts`, `list-query-aggregation.utils.ts` | Not modified |
| Audit-log modules, change-stream modules | Not modified |

### Files reviewed (2 changed)

| File | Change |
|------|--------|
| `src/controllers/edit-request-output.controller.ts` | OK/error message: “Sent for internal review” |
| `src/services/edit-request.service.ts` | Default crew note: “Sent for internal review” |

### DRY / KISS / Reusability / Global consistency scan

| Check | Verdict |
|-------|---------|
| Minimal string-only diff — no new abstractions | ✅ KISS |
| API response messages match new frontend toast/labels | ✅ |
| Persisted history notes on manager approve/changes still say “internal revision” | ❌ — see #1 |
| `edit-request-output.constants.ts` upload-hint strings unchanged (“internal revision”) | ⚠️ Out of PR diff; align in follow-up or expand scope |
| Protected list/audit modules untouched | ✅ |

### Positive notes

- **Surgical change:** Only updates the two user-visible default strings for crew send-to-manager; no layer or validation churn.
- **Cross-stack:** Matches frontend `appToast.success("Sent for internal review.")` and activity-history label `internal_revision_requested`.

---

## GitHub comments

### 1. `src/services/edit-request.service.ts` line 657

**PR comment (line 657):** **High (Global consistency):** This hunk renames the crew default note to “internal review”, but `reviewInternalRevision` (same file ~L788–797) still persists `"Manager approved internal revision"` and `"Manager requested changes on internal revision"`. Update those history defaults (and any matching version-event copy) in the same pass.

---

## Findings

---
Persisted history notes still use “internal revision” wording

Risk Level: HIGH  
**Status:** Open  
File Path: skillshow/src/services/edit-request.service.ts  
Lines: 788-797

Description:
**Global consistency.** This PR updates the crew default note and controller response to “internal review”, but `reviewInternalRevision` still writes history notes with legacy terminology:

```788:797:skillshow/src/services/edit-request.service.ts
        note: "Manager approved internal revision",
        ...
        note:
          body.note?.trim() || "Manager requested changes on internal revision",
```

Frontend `EDIT_REQUEST_HISTORY_TYPE_LABELS` and admin copy now display “internal review” for type labels, but **raw `note` fields** from the API will still surface “internal revision” in version notes, activity history detail lines, and any UI that shows the stored note verbatim.

Impact:
- Mixed terminology on the same request after manager approve/changes actions.
- Frontend normalization helpers cannot fix manager-authored default strings on new data.

Recommendation:
Update persisted defaults to match the rename:

```typescript
note: "Manager approved internal review",
// ...
note: body.note?.trim() || "Manager requested changes on internal review",
```

Scan sibling paths in `edit-request-output.service.ts` / version event logging for the same strings and update in the same PR for a complete migration.

**PR comment (line 657):** **High (Global consistency):** Extend this rename to `reviewInternalRevision` history defaults (~L788–797): `"Manager approved internal review"` and `"Manager requested changes on internal review"`.

---

## Summary

| # | Title | Risk | Status | File | Lines | PR comment line |
|---|--------|------|--------|------|-------|-----------------|
| 1 | Persisted history notes still use “internal revision” wording | HIGH | Open | skillshow/src/services/edit-request.service.ts | 788-797 | 657 |

**Merge readiness:** **Not merge-ready** — 1 open High (incomplete terminology migration on persisted history notes for **new** manager approve/changes actions). Frontend legacy note display ([#2](./frontend.md)) **Accepted**.
