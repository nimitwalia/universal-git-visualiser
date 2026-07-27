# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Universal Git Visualiser & Setup Studio — a zero-backend, fully client-side GitHub repository explorer. A user pastes an `owner/repo` string, the app fetches the repo tree directly from `api.github.com` from the browser, detects the project's ecosystem (Agentforce Skills, Salesforce DX, Node.js, Python, Docker, or generic), and hands back the exact copy-paste command to install/build/deploy it.

**The entire application is a single file: `index.html`.** There is no build step, no bundler, no package.json, and no test runner. All markup, styles, and ~1900 lines of vanilla JS logic live inside one `<script>` block starting at index.html:336.

## Running / testing locally

There is no build or lint tooling. To work on the app:

```bash
python3 -m http.server 8000   # or any static file server
# open http://localhost:8000
```

Then open the served `index.html` in a browser and exercise the feature manually (paste a repo like `forcedotcom/sf-skills` or `docker/getting-started-app`, click through file previews, diff mode, etc.). Testing in this repo is manual/exploratory — see `QA_REPORT.md` for the existing pass/fail matrix and reproduction steps used for the last verification round. There is no automated test suite to run.

## Hard architectural constraints

These are load-bearing project rules (from `v2-roadmap-agent-prompt.md`), not suggestions:

- **No backend, database, or persistent server-side state, ever.** Everything must run in the browser tab. GitHub API calls go straight from the client.
- **No new CDN-loaded runtime dependencies.** If a library is genuinely needed, download a minified build and vendor it directly in the repo rather than `<script src="https://cdn...">`. The existing Tailwind/marked.js/DOMPurify CDN `<script>` tags (index.html:12-16) are a known, already-flagged defect — do not add a second instance of the same problem by pulling in more CDN scripts.
- **Respect the unauthenticated GitHub rate limit (60 req/hr) as the default UX.** Any new feature that fires many API calls per user action needs to be scoped down (batch, cache, or make it opt-in behind a PAT) rather than burning through the limit in one use.
- **Match existing UI idioms** rather than inventing new ones: the "Copy Command" button + toast pattern, the two-row ribbon, the dark-panel command-card style used for install/deploy snippets.
- **After finishing a phase/feature, do a regression pass** on the core flow: ingest → detect ecosystem → preview file → copy command.

## Changelog workflow

`CHANGELOG.md` follows Keep a Changelog format, organized by phase (Phase 1 = quick wins, up through Phase 5 = diff viewer). When finishing a unit of work, draft the "Added"/"Fixed" entries in the same style as existing entries for the human to paste in — do not edit `CHANGELOG.md` unprompted. Also flag if `README.md`, `demo.gif`, or `intro-video-script.md` now describe the app inaccurately as a result of the change (e.g. new ecosystem detection changes the README's ecosystem list and the demo's narrated flow).

## Branches

- `main` — stable/published branch.
- `staging` — active development branch (current checkout). QA rounds and in-progress feature work happen here before merging to `main`.
- Live demo is deployed via GitHub Pages: https://nimitwalia.github.io/universal-git-visualiser/. `.nojekyll` at the repo root bypasses Jekyll processing, which previously caused Pages build timeouts — don't remove it.

## Code architecture (inside index.html's `<script>` block)

All state is module-level `let`/`const` globals declared right after the `<script>` tag opens (index.html:336-378) — there's no framework, no component model, no reactive state; DOM updates are done imperatively via direct `innerHTML`/`textContent` writes triggered from event handlers. Key globals: `currentOwnerRepo`, `repositoryTreeData`, `activeSignatures` (ecosystem detection flags), `expandedFolders`, `activeSelectedLines`/`lastClickedLineNum` (line-selection state), and the `isCompareMode`/`compareBaseText`/`changeBlocks` family (diff-viewer state).

The main data flow, roughly in call order:

1. **Ingestion** — `loadRemoteRepository()` (index.html:806) parses the `owner/repo` input, calls `fetchGitHubData()` to hit the GitHub REST API (repo metadata, then `git/trees/{branch}?recursive=1`), and flattens the response into `repositoryTreeData`.
2. **Ecosystem detection** — `detectManifestSignaturesFromTree()` (index.html:881) scans file paths for signature files (`skills/**/skill.md`, `sfdx-project.json`, `package.json`, `requirements.txt`/`pyproject.toml`, `Dockerfile`) and sets the `activeSignatures` flags plus renders ecosystem badges.
3. **Tree rendering** — `renderNestedTree()` (index.html:926) builds the collapsible file tree UI; `toggleFolder()`/`setAllFoldersState()` manage expand/collapse.
4. **File preview** — `selectSkill(path)` (index.html:1839) fetches a file's raw content, renders Markdown via `marked` + `DOMPurify.sanitize()` (never render raw Markdown unsanitized), or renders code via `renderLineNumberedCode()` → `tokenizeSyntax()` (a hand-rolled, zero-dependency single-pass regex highlighter — deliberately not Prism/Highlight.js, per the no-new-CDN-deps rule).
5. **Ecosystem action cards** — `renderCompositeSidebarWidgets()` (index.html:2040) generates the "Copy Command" cards (`npx skills add ...`, `sf project deploy ...`, per-script `npm run ...` buttons, Python venv/poetry setup, Docker commands) based on `activeSignatures` and the selected file.
6. **Manifest/security linting** — `validateAndScanManifest()` (index.html:1993) flags missing required fields in `sfdx-project.json`/`skill.json`, unpinned/`*`/`latest` npm dependency versions, `http:`/`git+http:` dependency URLs, pre/postinstall lifecycle scripts, and pipe-to-shell patterns in Dockerfiles. `checkNpmOsvVulnerabilities()` (index.html:1932) separately batches exactly-pinned npm deps to the OSV.dev API for known-CVE lookups (best-effort, fails silently offline).
7. **Diff viewer** — `myersDiff()` (index.html:1124) is a from-scratch Myers diff implementation comparing the active file against a historical commit SHA; `renderUnifiedDiffHtml()`/`renderSplitDiffHtml()` render the two view modes, and `navigateDiff()`/`initDiffNavigation()` handle jumping between contiguous change blocks (grouped via `data-diff-index` attributes, not DOM sibling walks — a prior bug source).
8. **Deep links** — `checkUrlDeepLinks()`/`parseLineHash()` support `?repo=owner/repo&path=...#L8-L11` style URLs (must accept both single-`L` and double-`L` GitHub-style range formats).

## GitHub token handling

A user-supplied fine-grained PAT (to raise the rate limit past 60/hr) is stored in `sessionStorage` (`universal_git_pat`) by default, cleared on tab close. Only if the user explicitly checks "Remember token on this device" does it get mirrored into `localStorage` (`universal_git_pat_persistent` flag + `universal_git_pat`). Token-related handlers live around index.html:396-530 (`loadTokenState`, `handleTokenInput`, `handleRememberMeToggle`, `executeTokenWipePurge`). Preserve this session-first/opt-in-persistence model when touching this code — it's a stated privacy guarantee in `README.md`.

## Other repo contents worth knowing about

- **`.github/workflows/sync-skills.yml`** — a daily cron job that polls `forcedotcom/sf-skills` and writes results to `data.json`. Note: `data.json` is **not** currently read or referenced anywhere in `index.html` — it appears to be an artifact from an earlier/parallel feature. Don't assume it's wired into the live app without checking.
- **`QA_REPORT.md`** — the authoritative record of the last manual verification pass (feature matrix + root-cause/fix writeups for prior bugs). Useful for understanding *why* certain code shapes exist (e.g. why the diff navigation uses `data-diff-index` instead of DOM sibling checks).
- **`v2-roadmap-agent-prompt.md`** — the phased build prompt used to drive prior feature work; still useful as a reference for the project's constraints and priorities if extending the app further.
