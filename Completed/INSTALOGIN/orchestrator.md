# Orchestrator PR Review — skillshow-distribution-orchestrator (`INSTALOGIN`)

**Repo:** SkillshowFx/skillshow-distribution-orchestrator  
**PR:** https://github.com/SkillshowFx/skillshow-distribution-orchestrator/pull/30  
**Branch:** `feat/insta-login` → main  
**Head:** `851e6909029eb4801843b2114fd1a319571fb435`  
**Scope:** Switch Instagram Graph host to `graph.instagram.com`, bump API version, prefer `metadata.accountId`, in-memory test harness refactor  
**Prompt:** `pr-review/prompts/orchestrator-system-prompt.md`  
**Paired:** `pr-review/Completed/INSTALOGIN/backend.md` (#260), `pr-review/Completed/INSTALOGIN/frontend.md` (#372)  
**Updated:** 2026-08-07 — archived (`.env.dev` finding accepted by reviewer)

## GitHub comments

_(none — no Open Critical/High/Medium findings)_

## Findings

---
Live credentials in tracked `.env.dev`

Risk Level: CRITICAL
File Path: .env.dev
Lines: 12, 28-29, 47-48, 77

Description:
**Security.** Head `851e690` partially addressed the finding: OAuth client IDs/secrets cleared, active `MONGODB_URI` emptied, and `.env.dev` added to `.gitignore`. Remaining exposure in diff (commented Mongo password, email app password, AWS keys, `SERVER_TOKEN`) accepted by reviewer to close INSTALOGIN review cycle.

Impact:
- Residual credential exposure in PR diff/git history on branch — acknowledged and accepted.

Recommendation:
Rotate AWS keys, email app password, MongoDB passwords, and `SERVER_TOKEN` if not already done post-merge.
---

**Positive notes:** Instagram vendor code changes (`graph.instagram.com`, API v22, `accountId` resolution order) align with backend #260. In-memory test harness refactor is scoped appropriately. OAuth vendor env vars cleared on latest head.

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Live credentials in tracked `.env.dev` | CRITICAL | Accepted | .env.dev | 12, 28-29, 47-48, 77 |

**Merge readiness:** **Merge-ready** — review archived with finding #1 accepted.
