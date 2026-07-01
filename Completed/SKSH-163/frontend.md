# PR review (SKSH-163) — skillshow-admin-ui

PR: `https://github.com/SkillshowFx/skillshow-admin-ui/pull/321`  
Base: `main`  
Head: `sksh-163` @ `1b5d2725`

Prompt: `pr-review/prompts/frontend-system-prompt.md`

## GitHub comments

No inline comments — no Open Critical/High/Medium findings.

## Findings

No Critical, High, or Medium findings.

### Files reviewed

| File | Change |
|------|--------|
| `src/theme/adapter/antd.adapter.tsx` | Global `ConfigProvider` `form={{ requiredMark: false }}` |
| `src/pages/management/policy/edit.tsx` | Remove manual `*` on policy name/content labels |
| `src/pages/management/policy/new.tsx` | Remove manual `*` on policy name/type/content labels |
| `src/pages/adminEditRequest/components/admin-edit-request-crew-return-section.tsx` | Remove manual `*` on return-reason label |
| `src/pages/videoLibrary/detail/index.tsx` | Remove `requiredMark={isEditable}` override |

### DRY / KISS / Global consistency scan

| Check | Verdict |
|-------|---------|
| Single source of truth for required marks (`ConfigProvider`) | ✅ |
| Manual label asterisks removed where they duplicated Form marks | ✅ All 6 instances in repo addressed |
| `requiredMark={isEditable}` override removed (would re-enable marks when editing) | ✅ |
| Protected module (`antd.adapter.tsx`) | ✅ **Accepted** — SKSH-163 explicitly scopes global form required-mark behavior |
| Server-list / destructive-action / audit-log contracts | N/A — styling-only change |
| Validation still enforced via `rules` / `required` on `Form.Item` | ✅ Unchanged |

### Positive notes

- **KISS:** One global `requiredMark: false` on `ConfigProvider` is the right lever; matches the pattern already used on many onboarding/auth forms.
- **Global consistency:** PR removes every hard-coded label asterisk (`text-error` / `text-red-500`) in the codebase and drops the video-library conditional override that would have shown marks in edit mode.
- **No behavioral regression:** Required-field validation and error messages are unchanged; only visual asterisks are suppressed.

### Optional follow-up (not reported as findings)

- ~20 forms still pass redundant `requiredMark={false}` now that the adapter sets it globally. Safe to delete in a small cleanup PR; no user-visible difference.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| — | No Critical/High/Medium findings | — | — | — | — |

**Merge readiness:** No open Critical/High/Medium blockers.
