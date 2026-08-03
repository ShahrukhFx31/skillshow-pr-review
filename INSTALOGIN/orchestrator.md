# Orchestrator PR Review — skillshow-distribution-orchestrator (`INSTALOGIN`)

**Repo:** SkillshowFx/skillshow-distribution-orchestrator  
**PR:** https://github.com/SkillshowFx/skillshow-distribution-orchestrator/pull/30  
**Branch:** `feat/insta-login` → main  
**Head:** `da70dfce229a43ad5a82bbda8542f593ea0284f9`  
**Scope:** Switch Instagram Graph host to `graph.instagram.com`, bump API version, prefer `metadata.accountId`, in-memory test harness refactor  
**Prompt:** `pr-review/prompts/orchestrator-system-prompt.md`  
**Paired:** `pr-review/INSTALOGIN/backend.md` (#260), `pr-review/INSTALOGIN/frontend.md` (#372)

## GitHub comments

_(none — no Open Critical/High findings)_

## Findings

_No Critical or High findings in the orchestrator diff. Changes align with Instagram Login tokens (`graph.instagram.com`), `accountId` resolution order matches backend metadata from #260, and test migration to in-memory mocks is scoped to repository tests._

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| — | — | — | — | — | — |

**Merge readiness:** No open Critical/High blockers on the orchestrator PR. Coordinate merge with backend #260 and frontend #372; legacy Instagram tokens will fail publish until users reconnect (expected — surface via frontend `needsReconnect` eligibility).
