# PR review (SKSH-386) — skillshow-admin-ui

| Field | Value |
|-------|-------|
| Repo | `SkillshowFx/skillshow-admin-ui` |
| PR | [#334](https://github.com/SkillshowFx/skillshow-admin-ui/pull/334) |
| Branch | `SKSH-386` → `main` |
| Head | `39256d82b1b122e27c5c820f4c04af85927584c0` |
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

## GitHub comments

### `src/layouts/components/account-dropdown.tsx`
- **HIGH** — Header account-general query key diverges from Account General cache (line 68)

## Findings

```markdown
---
Header account-general query key diverges from Account General cache

Risk Level: HIGH
File Path: src/layouts/components/account-dropdown.tsx
Lines: 68-79

Description:
**Global consistency** / **DRY** — Header now queries `["account-general", id]` while this PR’s Account General page (and all `setQueryData` / invalidation sites still used there) keep `PROFILE_QUERY_KEYS.accountGeneral` → `["account-general"]`. On `/user/*`, both queries are enabled. After a profile-image upload, General updates the bare key + `userStore.avatar`, but the header can still hold a stale `["account-general", id]` entry. Its sync effect then sees a different pathname and calls `setUserInfo({ avatar: fromApi })`, overwriting the fresh upload with the stale URL.

Impact:
- Uploaded/changed profile photo can briefly (or persistently) revert in the header.
- Duplicate network fetches for the same endpoint on account routes.
- User-scoped key only on one consumer does not actually isolate cache; writers still target the old key.

Recommendation:
Prefer one of:
1. **Revert** the header to `queryKey: PROFILE_QUERY_KEYS.accountGeneral` — `clearClientQueryCache()` on sign-in/logout already clears cross-user pollution; or
2. **Migrate every touched reader/writer in this PR** (at least `src/pages/user/account/general/index.tsx` `useQuery` + all `setQueryData` for account-general) to the same `["account-general", userId]` shape, and invalidate/update that key on mutations.

Do not leave header and Account General on different keys.
---
```

PR comment (inline, 2–4 sentences):
Header now uses `["account-general", id]` while Account General still reads/writes `["account-general"]`. On `/user/*` both run, so a stale header cache can overwrite a freshly uploaded avatar via the sync effect. Either keep the shared `PROFILE_QUERY_KEYS.accountGeneral` key (cache clear on auth is enough) or migrate every reader/writer in this PR to the same user-scoped key.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Header account-general query key diverges from Account General cache | HIGH | Open | `src/layouts/components/account-dropdown.tsx` | 68-79 |

**Merge readiness:** Request changes — fix the account-general query-key split before merge. Pathname-based avatar sync and auth cache clearing look good.
