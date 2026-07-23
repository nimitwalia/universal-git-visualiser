# Changelog

All notable changes to Universal Git Visualiser & Setup Studio are documented here, grouped by build phase rather than by individual commit. Format loosely follows [Keep a Changelog](https://keepachangelog.com/).

Each phase entry should only be filled in **after** the phase has been built and verified working — not written in advance of the work landing. When a phase entry is added, check whether the README, `demo.gif`, and `intro-video-script.md` still accurately describe the app; if not, note that as a follow-up in the entry.

---

## [Unreleased]

Nothing pending yet.

---

## [Baseline] - 2026-07-15

Starting point this changelog picks up from. Covers everything shipped before phased v2 work began.

### Added
- Core ingestion flow: paste `owner/repo` or full GitHub URL, fetch full file tree client-side, no backend.
- Ecosystem detection for Agentforce Skills and Salesforce DX, with matching `npx skills add` / `sf project deploy` command cards.
- File preview pane with Markdown rendering (DOMPurify-sanitized) and raw code view.
- Client-side-only token handling (sessionStorage by default, opt-in localStorage persistence), with one-click purge.
- Copy Deep-Link feature (`?repo=owner/repo`).
- Two-row ribbon redesign, sentence-case badges, simplified command-card copy (21-item UI/UX cleanup pass).

### Fixed
- Critical: bad/remembered GitHub token causing indefinite freeze on reload (was TC-SEC-05) — replaced blocking native `prompt()` with in-page auth modal.
- Stale file-preview panel not resetting between repo switches.
- Enter-key not submitting the repo field.
- Node.js/Salesforce DX badge conflict (extra badge shown when it shouldn't be).
- Token purge leaving a leftover persistence flag in localStorage.

### Known open issues (not yet scheduled)
- Invalid-format validation (`TC-ING-04`) still uses a native `alert()` instead of the in-page modal pattern used elsewhere.
- Tailwind CSS still loaded from CDN — production console warning on every load.
- API-limit low-hint copy doesn't match spec wording exactly.

---

## [Phase 1] - TBD
*Quick wins: collapse/expand all, file-type icons, copy-raw-file button, repo health panel, language breakdown bar.*

### Added
-

---

## [Phase 2] - TBD
*Self-contained UX upgrades: syntax highlighting + line numbers, in-file search, deep-link to line ranges.*

### Added
-

---

## [Phase 3] - TBD
*Ecosystem & validation expansion: Node/NPM, Python, Docker detection; manifest validation warnings.*

### Added
-

### Follow-up needed
- README "How it works" section and `demo.gif` only cover Agentforce/Salesforce DX detection — update once this phase ships.

---

## [Phase 4] - TBD
*Higher-effort subsystems: pre-execution security lint, scoped in-session content search.*

### Added
-

---

## Deferred (not scheduled)
- Multi-tab preview canvas — deferred pending a separate UI-redesign decision.
- Full unscoped global repo content search — deferred pending a decision on making a token a hard requirement for this specific feature.
