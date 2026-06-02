# SKSH-277 — Backend review (`skillshow`)

**Branch:** `sksh-277`  
**Base:** `main`  
**Re-review:** 2026-06-01 (no backend changes since initial review)

## Overview

Parent dashboard athlete feed is scoped to videos uploaded **on or after** the link became `ACCEPTED`, via `linkedAt` on `LeanViewerRelationWithAthleteUser`, `linkedAthleteVideoScopes` in `ParentLinkService.getDashboard`, and `VideoRepository.findRecentByLinkedAthletesLean`.

Backend diff vs `main` is limited to 5 files (no formatting-only churn).

---

## Positive notes

- **Layering:** Repository maps `linkedAt`; service builds scopes; repository runs the Mongo query.
- **`.lean()` + projection** via `DASHBOARD_VIDEO_SELECT`; feed `limit` unchanged.
- **Index `{ user: 1, createdAt: -1 }`** supports each `$or` branch.
- **Tests** assert `findRecentByLinkedAthletesLean` receives per-athlete `{ userId, linkedSince }` scopes.

---

## Findings

No **Critical** or **High** findings.

---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|

*No Critical/High findings.*
