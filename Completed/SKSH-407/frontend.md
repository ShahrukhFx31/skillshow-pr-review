# PR review (SKSH-407) — skillshow-admin-ui

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow-admin-ui` |
| PR | [#347](https://github.com/SkillshowFx/skillshow-admin-ui/pull/347) |
| Branch | `SKSH-407` → `main` |
| Head | `68074b2c8ac53297fe061c9c8950c1836de8aa9b` |
| Scope | Teams list loading: `isPending`/`isFetching`, skeleton only on initial load, card motion |
| Prompts | `pr-review/prompts/frontend-system-prompt.md` |

### Protected modules

| Module | Status |
|--------|--------|
| Pagination / table / audit protected modules | **Not modified** ✅ |

### Positive notes

- `isInitialLoading = isPending && teamsPage === undefined` correctly preserves `keepPreviousData` rows during refetch.
- Empty states wait for `!isFetching`, avoiding flicker to empty during search/page changes.
- Skeletons only on true first load; `TeamListCard` `initial={false}` reduces remount animation noise.

## GitHub comments

No open GitHub inline comments.

## Findings

No reportable Critical, High, or Medium findings.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| — | No reportable findings | — | — | — | — |

**Merge readiness:** Ready — no open Critical/High/Medium blockers.
