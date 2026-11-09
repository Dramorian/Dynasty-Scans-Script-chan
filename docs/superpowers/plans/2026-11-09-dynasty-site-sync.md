# Dynasty-Scans Script-chan Site Sync Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Bring the Dynasty-Scans Script-chan userscript up to date with the current dynasty-scans.com (rebuilt ~6 years after the script was last touched), fixing verifiable breakages and refreshing metadata so Tampermonkey auto-update works.

**Architecture:** Single-file userscript (`Dynasty-Scans Script-chan.user.js`). Changes are surgical: fix the tags.json parsing in `createTagMap()`, re-point the mangadex searcher at the current heading-based page titles, and bump the script header (version + repo URLs). No new dependencies; no restructure — the file is small enough to hold in context and the codebase uses one large file.

**Tech Stack:** Vanilla ES6 + jQuery userscript (Tampermonkey/Greasemonkey), `@require jquery.hotkeys`.

**Spec:** User request: "Make it up to date with the site. The script was last updated 6 years ago." Recon performed against live dynasty-scans.com (home, chapter, issue, forum, images, tags.json).

## Global Constraints
- Conventional Commits for every commit (`feat:`, `fix:`, etc.).
- Do not break features that still work; only change selectors confirmed (or strongly indicated by live-page markup) to have changed.
- Keep `@require` jQuery.hotkeys intact.
- Version must be incremented for Tampermonkey auto-update to trigger.

## Review Focus
1. `tags.json` now returns `{"tags":[{...}],"current_page":1,"total_pages":4}` — `createTagMap()` must read `data.tags`, not `Object.values(data)` (which yields numbers and throws).
2. Chapter/issue pages no longer have `#chapter-title`/`.tag-title`; titles are now heading-based (`### **Title**` / `## **Title**`) — mangadex selectors must target headings or the feature stays broken.
3. Other CSS-selector features (navbar, pagination, spoilers, BBCode, stats, etc.) could not be verified against live HTML (no raw-DOM access) — left untouched unless clearly broken.

## Tasks
- [ ] **Task 1: Fix tags.json parsing in `createTagMap()`**
  - **Files:** Modify: `Dynasty-Scans Script-chan.user.js` (`createTagMap`)
  - **Interfaces:** `createTagMap()` consumes `$.getJSON(tagsJSON)`; produces a tagMap keyed by lowercase tag name.
  - [ ] Step: Edit `createTagMap()` to read `data.tags` with fallback to `Object.values(data)` for old format.
  - [ ] Step: `node --check` syntax validation.
  - [ ] Step: Commit `fix: handle tags.json pagination wrapper in forum tagger`.

- [ ] **Task 2: Re-point mangadex searcher at current page titles**
  - **Files:** Modify: `Dynasty-Scans Script-chan.user.js` (`mangadex`, `getWorkInfo`)
  - **Interfaces:** `getWorkInfo("title"/"author")` returns work title/author strings from chapter/issue/series/anthology pages.
  - [ ] Step: Update `select` insertion target and `getTitle`/`getAuthors` to target heading-based titles (`h2`/`h3` `strong`) with author links.
  - [ ] Step: `node --check` syntax validation.
  - [ ] Step: Commit `feat: retarget mangadex searcher at rebuilt page titles`.

- [ ] **Task 3: Refresh script metadata and bump version**
  - **Files:** Modify: `Dynasty-Scans Script-chan.user.js` header block; `README.md` (version/repo URLs).
  - **Interfaces:** `@version`, `@website`, `@downloadURL`, `@updateURL`.
  - [ ] Step: Bump `@version` 0.1 → 0.2.0; point website/downloadURL/updateURL at Dramorian/Dynasty-Scans-Script-chan.
  - [ ] Step: Commit `chore: update script metadata and bump version for auto-update`.

- [ ] **Task 4: Final verification and commit**
  - **Files:** `Dynasty-Scans Script-chan.user.js`, `README.md`
  - [ ] Step: Run `node --check` on the final script; review `git diff`.
  - [ ] Step: Commit any remaining changes; present summary to user.
