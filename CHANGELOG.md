# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [v2.0.0-staging] - 2026-07-24

### Added
- **Phase 1 (Quick Wins):**
  - Directory Tree bulk expand/collapse all toggle controls.
  - Zero-dependency file extension glyph lookup table.
  - Preview header "Copy Raw" content button with toast notification feedback.
  - At-a-Glance Repository Health strip displaying commit date, workflow detection, license, and open issue counts.
  - Client-side computed architecture distribution progress bar.
- **Phase 2 (Self-Contained UX):**
  - Zero-CDN client-side syntax tokenizing engine for JS, TS, JSON, Python, Apex, XML, and YAML files.
  - In-file match search (`Cmd+F`) with match highlighting inside the preview pane.
  - Deep-link support for line fragments (`#L12-L25`) with smooth auto-scroll and line highlighting.
- **Phase 3 (Ecosystem & Validation Expansion):**
  - Dynamic Node.js package script parser listing all defined `package.json` scripts.
  - Python ecosystem disambiguation surfacing virtual environment setup (`python -m venv`) for `requirements.txt` and `poetry install` for `pyproject.toml`.
  - Docker and Docker Compose environment setup command card generation.
  - Client-side manifest validation warnings for SFDX and Agentforce skill manifests.
- **Phase 4 (Security & In-Session Search):**
  - Pre-execution supply-chain security linter flagging lifecycle scripts, pipe-to-shell patterns, unpinned dependency versions, and non-standard registry URLs.
  - Scoped in-session content search across opened files with instant match navigation.
- **Phase 5 (Version Comparison & Diff Viewer) [NEW]:**
  - Client-side Myers Diff comparison engine comparing active file previews against historical GitHub commits.
  - Side-by-Side (Split) and Unified diff views with responsive rendering.
  - Dynamic full-width compare canvas mode: Automatically collapses the right actions column on entering compare mode to maximize horizontal space.
  - Inline text word-wrapping (`whitespace-pre-wrap break-all`) to prevent horizontal scrollbars.
  - Sequential contiguous change block pairing in Split view for cleaner side-by-side line alignment.
  - Floating settings bar (`sticky top-[60px]`) containing view settings, Next/Prev navigation buttons, and boundary locking indicators that remain stationary during page scroll.
  - Added SVG favicon (`favicon.svg`) styled after the indigo "UNIV" header badge.
  - Added known-CVE lookup for pinned npm dependencies via the free OSV.dev API (batched, best-effort, fails silently if unreachable) — closes a gap where an exactly-pinned-but-vulnerable dependency previously passed the security linter with no warning.



### Fixed
- Fixed race condition in `processLineFragmentSelection` where line highlights were cleared on re-render.
- Fixed single-script truncation in `package.json` output to surface all available project scripts.
- Fixed in-session search to auto-open and navigate to matching file previews on search execution.
- Fixed compare navigation page-scroll calculations to eliminate target overshooting.
- Fixed diff text contrast against dark editor background by introducing default `text-slate-100` and removed `text-yellow-100` styles.
- Fixed GitHub Actions deployment timeouts by introducing a `.nojekyll` bypass file to the staging repository root.
- Fixed syntax highlighter tag corruption on non-JSON files by converting `tokenizeSyntax` to single-pass combined regex.
- Fixed diff navigation on large files by grouping change blocks using sequential `data-diff-index` row dataset attributes instead of DOM sibling checks.
- Fixed deep-link range parsing in `parseLineHash` to support GitHub-style double-L URL formats (`#L8-L11`).
- Fixed gutter line selection so plain Shift-click is always additive and preserves prior Cmd/Ctrl-click selections.
- Fixed `navigateDiff()` visibility guard (Bug A) to prevent unnecessary page jumps when target change block is already visible in viewport.
- Fixed `navigateDiff()` smooth-scroll race conditions (Bug B) using `requestAnimationFrame` layout settlement and scroll position diagnostics.



