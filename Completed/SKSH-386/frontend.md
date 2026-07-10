# PR review (SKSH-386) — skillshow-admin-ui

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow-admin-ui` |
| PR | [#334](https://github.com/SkillshowFx/skillshow-admin-ui/pull/334) |
| Branch | `SKSH-386` → `main` |
| Head | `4895f612f4efcf9ab4c16c8a742de0689ed3db58` |
| Scope | Stop header avatar flicker from rotating presigned URLs; shared `queryClient` + clear on auth boundaries; header avatar error fallback |
| Prompts | `pr-review/prompts/frontend-system-prompt.md` |

### Protected modules

| Module | Status |
|--------|--------|
| `pagination-bar.tsx`, `use-pagination.ts`, `use-server-table-controls.ts`, `table-sort.ts`, audit-log UI stack, `antd.adapter.tsx` | **Not modified** ✅ |

### Positive notes

- `getProfileImageSyncKey` / `shouldUpdateStoredProfileAvatar` correctly ignore S3 query-string churn — root cause of the delay/flicker.
- Shared `queryClient` + `clearClientQueryCache` on login, logout, refresh failure, and 401 is the right auth-boundary hygiene.
- `AccountHeaderAvatar` keeps load failures local (no store thrash).
- `setUserInfo({ avatar })` is safe: store merges partial patches.
- Header and Account General now share `PROFILE_QUERY_KEYS.accountGeneral`.

## GitHub comments

_No open findings._

## Findings

---
Header account-general query key diverges from Account General cache

Risk Level: HIGH
File Path: src/layouts/components/account-dropdown.tsx
Lines: 72-75

Description:
**Global consistency** / **DRY** — Header briefly queried `["account-general", id]` while Account General kept `PROFILE_QUERY_KEYS.accountGeneral` → `["account-general"]`. On `/user/*`, both ran; a stale header cache could overwrite a freshly uploaded avatar via the sync effect.

Impact:
- Uploaded/changed profile photo could revert in the header.
- Duplicate network fetches for the same endpoint on account routes.

Recommendation:
Keep header on `PROFILE_QUERY_KEYS.accountGeneral` (auth cache clear is enough), or migrate every reader/writer to the same user-scoped key.

**Re-verify:** ✅ Fixed — header uses `queryKey: PROFILE_QUERY_KEYS.accountGeneral`, matching Account General readers/writers.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Header account-general query key diverges from Account General cache | HIGH | ✅ Fixed | `src/layouts/components/account-dropdown.tsx` | 72-75 |

**Merge readiness:** No open Critical/High/Medium blockers.
