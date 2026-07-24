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

### Fixed
- Fixed race condition in `processLineFragmentSelection` where line highlights were cleared on re-render.
- Fixed single-script truncation in `package.json` output to surface all available project scripts.
- Fixed in-session search to auto-open and navigate to matching file previews on search execution.
