# PR review (SKSH-402) — skillshow

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow` |
| PR | [#243](https://github.com/SkillshowFx/skillshow/pull/243) |
| Branch | `SKSH-402` → `main` |
| Head | `d83eee1dcb16016ad8f0398c897593b33c44965f` |
| Scope | Crew list fields (otherRoles / availability); invite `accountLabel`; export columns |
| Prompts | `pr-review/prompts/backend-system-prompt.md`, `pr-review/SECURITY-AUDIT-PRE-RELEASE.md` |
| Verified | 2026-07-17 — head unchanged; still ready |

### Protected modules

| Module | Status |
|--------|--------|
| List-query / audit / change-stream protected modules | **Not modified** ✅ |

### Positive notes

- New fields projected in `LIST_ROW_PROJECT` and typed on `CrewUserListRow`.
- Sort keys added to `CREW_USER_LIST_SORT_FIELD_MAP` (allow-list stays in sync via `Object.keys`).
- Invite `accountLabel` defaults to `"team member"` for backward compatibility; crew passes `"Crew"`.

## GitHub comments

No open GitHub inline comments.

## Findings

No reportable Critical or High findings.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| — | No reportable findings | — | — | — | — |

**Merge readiness:** Ready — no open Critical/High blockers.
