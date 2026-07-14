# PR review (SKSH-405) — skillshow-admin-ui

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow-admin-ui` |
| PR | [#338](https://github.com/SkillshowFx/skillshow-admin-ui/pull/338) |
| Branch | `SKSH-405` → `main` |
| Head | `61b95ca9be8f295737cc11536028533073e90961` |
| Scope | Stricter email validation for partner onboarding (`EMAIL_PATTERN`, `isValidAppEmail`, contact rules) |
| Prompts | `pr-review/prompts/frontend-system-prompt.md` |

**Aligned with:** [backend.md](./backend.md)

### Protected modules

| Module | Status |
|--------|--------|
| pagination-bar, use-pagination, use-server-table-controls, table-sort, audit-log stack, antd.adapter | **Not modified** ✅ |

### Positive notes

- Single validation source via `isValidAppEmail` / `EMAIL_PATTERN`, with sync note for backend `PARTNER_CONTACT_EMAIL_PATTERN`.
- `partnerContactEmailRules()` + shared message constants avoid copy-pasted Ant `rules`.
- Submit path re-checks emails and lowercases after trim.

## GitHub comments

_No open findings._

## Findings

_No Critical, High, or Medium findings._

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| — | _(none)_ | — | — | — | — |

**Merge readiness:** No open Critical/High/Medium blockers.
