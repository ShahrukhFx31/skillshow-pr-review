# PR review (SKSH-400) — skillshow-admin-ui

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow-admin-ui` |
| PR | [#344](https://github.com/SkillshowFx/skillshow-admin-ui/pull/344) |
| Branch | `SKSH-400` → `main` |
| Head | `78284f315cce9c3c4276866b8588e5117c77d8dd` |
| Scope | Email notification settings: debounced coalesce + optimistic cache; per-key loading; selective error rollback |
| Prompts | `pr-review/prompts/frontend-system-prompt.md` |
| Re-verify | 2026-07-16 — head `78284f31`; both prior MEDIUMs fixed |

### Protected modules

| Module | Status |
|--------|--------|
| Pagination / table control protected modules | **Not modified** ✅ |
| Audit-log protected modules | **Not modified** ✅ |
| `destructive-action-confirm-modal.tsx` | **Consumed only** (visibility confirm) ✅ |

### Positive notes

- Debounce coalesce batches rapid multi-toggle flips into one PATCH.
- Only in-flight patch keys are disabled/loading; siblings stay interactive.
- Mid-save sibling edits are preserved on success and flushed by the next debounce cycle.
- `inFlightPrefKeysRef` enables partial error rollback without wiping unrelated toggles.
- Cancelling in-flight GETs + optimistic `setQueryData` addresses toggle flicker.

## GitHub comments

No open GitHub inline comments.

## Findings

---
Serialized preference saves remove prior multi-toggle batching

Risk Level: MEDIUM
File Path: src/pages/user/account/settings-tab.tsx
Lines: prior

Description:
**KISS / UX contract** — Earlier head fired a PATCH on the first change and blocked siblings via a single `updatingPrefKey`.

**Re-verify (d9adf53e / 78284f31):** ✅ Fixed — 400ms debounce + `diffEmailPrefs` batching; per-key `updatingPrefKeys` only.
---

---
Error rollback wipes sibling toggles made during in-flight save

Risk Level: MEDIUM
File Path: src/pages/user/account/settings-tab.tsx
Lines: 87-112

Description:
**Correctness** — Earlier head reset all local prefs/cache on error.

**Re-verify (78284f31):** ✅ Fixed — `inFlightPrefKeysRef` records patch keys; `onError` rolls back only those keys in local state and query cache; sibling mid-flight diffs remain for debounce retry.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Serialized preference saves remove prior multi-toggle batching | MEDIUM | ✅ Fixed | `src/pages/user/account/settings-tab.tsx` | prior |
| 2 | Error rollback wipes sibling toggles made during in-flight save | MEDIUM | ✅ Fixed | `src/pages/user/account/settings-tab.tsx` | 87-112 |

**Merge readiness:** Ready — no open Critical/High/Medium blockers.
