# PR review (SKSH-371) — skillshow-admin-ui
PR: `https://github.com/SkillshowFx/skillshow-admin-ui/pull/317`
Base: `main`
Head: `SKSH-371`

Prompt: `pr-review/prompts/frontend-system-prompt.md`

## GitHub comments

### `src/pages/user/account/connections/hooks/use-connection-relation-deep-link.ts`
- **MEDIUM** — Deep-link param can get “stuck” on invalid relationId (line ~33)

## Findings

```markdown
---
Deep-link param can get “stuck” on invalid `relationId`

Risk Level: MEDIUM
File Path: src/pages/user/account/connections/hooks/use-connection-relation-deep-link.ts
Lines: 29-44

Description:
`useConnectionRelationDeepLink` only clears `?relationId=` when the verification call returns 404. If the param is malformed/invalid (often a 400) or access is denied (403), the URL keeps the param but no row can ever highlight, and `verifiedIdsRef` prevents retry (so the user can’t recover without manually editing the URL).

Impact:
- Confusing UX for deep links from notifications/emails when the relationId is invalid or temporarily unauthorized.
- Support/debug friction because the page state doesn’t self-heal.

Recommendation:
Treat “definitely not highlightable” cases as clearable too:
- Clear on 400 (invalid id / validation errors).
- Consider clearing on 403 only if product expects “no access” to behave like “not found”; otherwise keep it but allow a retry by not caching failures (move `verifiedIdsRef.add()` after a successful `getRelation` or track success separately).
---
```

PR comment (inline, 2–4 sentences):
`useConnectionRelationDeepLink` only clears `?relationId=` on 404. If the param is malformed (400) or temporarily forbidden (403), the URL can get “stuck” with an un-highlightable relationId and `verifiedIdsRef` prevents any retry. Consider clearing on 400 and only caching verified IDs after a successful lookup (or track success/failure separately).

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Deep-link param can get “stuck” on invalid `relationId` | MEDIUM | Open | `src/pages/user/account/connections/hooks/use-connection-relation-deep-link.ts` | 29-44 |

**Merge readiness:** No open Critical/High blockers (1 Medium Open).
