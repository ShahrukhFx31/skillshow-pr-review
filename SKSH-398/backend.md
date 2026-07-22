# PR review — skillshow #249 (SKSH-398)

**Repo:** SkillshowFx/skillshow  
**Branch:** SKSH-398 → main  
**Head:** `c195da1839cd9a9cbd1fbcc3b453887f6fba8e05`  
**Scope:** Admin/super_admin fallback in `findOwnedVideoKey` for shared Video Library playback  
**Prompt:** `pr-review/prompts/backend-system-prompt.md`

## GitHub comments

### `src/services/video.service.ts`

- **L536** — Admin library playback filter should reuse `VIDEO_LIBRARY_LIST_MATCH`

## Findings

---
Admin library playback filter should reuse VIDEO_LIBRARY_LIST_MATCH

Risk Level: HIGH
File Path: src/services/video.service.ts
Lines: 536-545

Description:
**DRY / Global consistency / Contract.** The admin fallback hand-builds `{ inVideoLibrary, libraryStatus, uploadStatus }` while the established library match is `VIDEO_LIBRARY_LIST_MATCH` in `video-library.constants.ts` (also mirrored by `VIDEO_SKILL_SHOW_TAGGED_MATCH`). That shared match additionally requires `librarySeq: { $exists: true }`. Omitting it lets admins resolve play keys for incomplete / non-sequenced library-flagged rows that list/get APIs would not treat as library rows — and the two filters can drift further over time.

**Security:** Expanding play-url to admins for shared library rows (after own-scope miss) is appropriate; `hasAnyRole(..., ["admin"])` correctly includes `super_admin` via `expandAdminRoleRequirement`. Not an IDOR for non-admins.

Impact:
- Admins can obtain playback keys for rows that are not yet (or never) valid library entries (`librarySeq` missing).
- Future library visibility rules updated only in `VIDEO_LIBRARY_LIST_MATCH` will not apply here.

Recommendation:
Reuse the shared match:

```ts
import { VIDEO_LIBRARY_LIST_MATCH } from "../constants/video-library.constants";

// ...
const library = (await videoRepository.findOneLeanByFilter(
  {
    _id: videoId,
    ...VIDEO_LIBRARY_LIST_MATCH,
  },
  "key"
)) as { key: string } | null;
```

Drop the standalone `VIDEO_LIBRARY_DELETED_STATUS` import if unused.
---

## Summary

| # | Title | Risk | Status | File | Lines |
|---|--------|------|--------|------|-------|
| 1 | Admin library playback filter should reuse VIDEO_LIBRARY_LIST_MATCH | HIGH | Open | src/services/video.service.ts | 536-545 |

**Merge readiness:** Request changes — open High finding #1.
