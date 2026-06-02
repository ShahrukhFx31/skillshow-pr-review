# Orchestrator PR Review — skillshow-distribution-orchestrator (`SKSH-267`)

**Base:** `main...HEAD`  
**Branch:** `sksh-267`  
**Re-verified:** 2026-05-25  
**Changed files:** `vendorLog.model.ts`, `VendorLogRepository.ts`, `VideoProcessWorker.ts`, tests  
**Findings:** 1 reviewed — **1 fixed**, **0 open**

---

## `attempts` default change without data migration

**Status:** ✅ **FIXED** (manual backfill — not in repo)

**Risk Level:** HIGH  
**File Path:** `src/models/vendorLog.model.ts`  
**Lines:** 34-38

**Original concern:**  
Schema default for `attempts` changed to `1`; legacy documents with `attempts: 0` or missing field could show inconsistent counts in admin UI.

**Verification:**  
No migration script in the repository. Developer confirmed **manual backfill** was run against the database (ops/deploy step), so legacy vendor logs align with the new semantics (`attempts` = user-initiated distribute/retry cycles, not BullMQ redeliveries).

**Note for reviewers:** Consider documenting the manual script/steps in runbook or ticket comments for reproducibility in other environments (staging/production).

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | `attempts` default without migration | HIGH | ✅ Fixed (manual DB backfill) | `src/models/vendorLog.model.ts` | 34-38 |

## Positive notes

- `markProcessing` no longer increments `attempts` on BullMQ pickup — aligns with manual retry via `resetForRetry`
- Tests added for `resetForRetry` increment and `markProcessing` preserving attempts
- Comments in repository clarify attempts semantics

**All reported orchestrator issues are resolved on the current branch.**

---

## Regression re-check (2026-05-25)

**No new Critical or High issues introduced** by the `attempts` semantics change (`markProcessing` no longer increments, `resetForRetry` increments, default `attempts: 1`, worker creates with `attempts: 1`). Tests cover `markProcessing` and `resetForRetry` behavior.

| Result | Notes |
|--------|--------|
| ✅ No new findings | Layering, worker/repository changes, and poll/retry semantics look consistent |
| ℹ️ Out of scope | `.env.dev` credential diff (not evaluated in this pass) |
