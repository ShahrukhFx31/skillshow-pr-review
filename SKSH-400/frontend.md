# PR review (SKSH-400) — skillshow-admin-ui

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow-admin-ui` |
| PR | [#344](https://github.com/SkillshowFx/skillshow-admin-ui/pull/344) |
| Branch | `SKSH-400` → `main` |
| Head | `4c103a65f22ee18fd1ca8a8cdf8566a10c486207` |
| Scope | Email notification settings: remove debounce auto-save; optimistic per-key PATCH; flicker guard |
| Prompts | `pr-review/prompts/frontend-system-prompt.md` |

### Protected modules

| Module | Status |
|--------|--------|
| Pagination / table control protected modules | **Not modified** ✅ |
| Audit-log protected modules | **Not modified** ✅ |
| `destructive-action-confirm-modal.tsx` | **Consumed only** (visibility confirm) ✅ |

### Positive notes

- Cancelling in-flight `profileSettings` GETs and optimistically updating the cache correctly addresses the off→on→off toggle flicker.
- Guarding server→local sync while `updatingPrefKey !== null` prevents stale refetches from overwriting the optimistic value mid-save.
- Error path rolls back both local prefs and query cache, then clears the in-flight key.
- Per-switch `loading={updatingPrefKey === key}` is a clear UX improvement over spinning every toggle.

## GitHub comments

### `src/pages/user/account/settings-tab.tsx`

- **L85** — Preference saves are fully serialized; multi-toggle batching removed (MEDIUM)

## Findings

---
Serialized preference saves remove prior multi-toggle batching

Risk Level: MEDIUM
File Path: src/pages/user/account/settings-tab.tsx
Lines: 85-102

Description:
**KISS / UX contract** — The debounce + `diffEmailPrefs` path previously let the user flip several toggles within ~400ms and send one PATCH. This PR fires a PATCH on the first change and then blocks further changes with `updatingPrefKey !== null` (and `disabled={isUpdating}` on every switch). Sibling prefs can no longer be edited until the in-flight save settles.

Impact:
- Changing multiple email preferences requires N sequential round trips instead of one batched save.
- Other toggles appear stuck/disabled while a single preference is saving.

Recommendation:
Keep optimistic updates and the mid-save sync guard (they fix the flicker), but restore a short coalesce window or allow concurrent per-key updates:

```ts
// Option A: per-key pending map; don't block siblings
if (localEmailPrefs[key] === value || pendingKeys.has(key)) return;

// Option B: keep a short debounce that PATCHes Partial<EmailNotificationPrefs>
// after the last flip, while still cancelQueries + optimistic setQueryData
```

Only disable/load the switch that is saving; leave siblings interactive.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Serialized preference saves remove prior multi-toggle batching | MEDIUM | Open | `src/pages/user/account/settings-tab.tsx` | 85-102 |

**Merge readiness:** No open Critical/High blockers; 1 Medium open (multi-toggle batching / sibling lock).
