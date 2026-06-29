# PR review (SKSH-371) — skillshow
PR: `https://github.com/SkillshowFx/skillshow/pull/220`
Base: `main`
Head: `SKSH-371`

Prompt: `pr-review/prompts/backend-system-prompt.md`

## GitHub comments

### `src/constants/notification-platform.constants.ts`
- **HIGH** — EditRequest email rate limiting loosened significantly (line ~76)

## Findings

```markdown
---
EditRequest email rate limiting loosened significantly

Risk Level: HIGH
File Path: src/constants/notification-platform.constants.ts
Lines: 76-85

Description:
The `editRequest` domain policy increases `userPerHour` from 20 → 200 and removes `perKindMinutes`. This is a large behavioral change that can materially increase outbound email volume for a single user in a short period, especially during rapid workflow transitions.

Impact:
- Increased chance of email spam / user annoyance for edit-request flows.
- Higher provider cost and potential deliverability impact if a bug causes event storms.

Recommendation:
If this change is required for correctness, tighten the blast radius:
- Keep the higher `userPerHour` but add a smaller scoped limiter (e.g. per `editRequestId` / `scopeId`) for high-frequency kinds, or explicitly document which kinds are exempt.
- Add a metric/log to alert on unusually high per-user send counts for `editRequest` to catch regressions quickly.
---
```

PR comment (inline, 2–4 sentences):
This bumps `editRequest` email limits from 20→200 per user/hour and drops `perKindMinutes` entirely. That’s a big behavioral shift and could amplify email volume during rapid workflow transitions or event storms. If correctness requires removing `perKindMinutes`, consider adding a smaller scoped limiter (e.g. per `editRequestId`) and alerting/metrics for per-user send spikes.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | EditRequest email rate limiting loosened significantly | HIGH | Open | `src/constants/notification-platform.constants.ts` | 76-85 |

**Merge readiness:** Open High blocker (rate limiting change needs explicit justification/mitigation).
