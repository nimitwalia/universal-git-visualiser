# Universal Git Visualiser — v2 Build Prompt for Coding Agent

Copy everything below the line into your coding agent as the task prompt. It's phased by effort, not by importance — Phase 1 items are small and self-contained; by Phase 4 you're touching detection logic and new subsystems, so budget more review time there.

---

## Context for the agent

You're extending **Universal Git Visualiser & Setup Studio**, a zero-backend, fully client-side GitHub repository explorer. It scans a repo, detects its ecosystem (Agentforce Skills, Salesforce DX, and soon Node/Python/Docker), and hands the user the exact copy-paste command to run, build, or deploy it — without cloning. Everything runs in the browser tab; there is no server component, and that architectural constraint is not negotiable — do not introduce anything that requires a backend, a database, or persistent server-side state.

**Hard constraints for every phase below:**
- No new CDN-loaded runtime dependencies. If a library is needed (e.g. a syntax highlighter), download a minified build and bundle it locally in the repo — do not `<script src="https://cdn...">` it. This app already has one open bug (Tailwind loaded from CDN) flagged as a defect in QA; do not add a second instance of the same problem.
- Respect the existing unauthenticated rate limit (60 req/hr) as the default experience. Any feature that would require many new API calls per action needs to be scoped down to avoid exhausting it in a single use — see Phase 4 notes.
- Match the existing UI patterns already in place (the "Copy Command" button + toast pattern, the two-row ribbon, the dark-panel command-card style) rather than introducing new visual idioms.
- After each phase, run a regression pass against the existing ingestion → detect → preview → copy-command flow before moving to the next phase.

**End-of-phase deliverable — required every phase:**
This repo tracks changes in `CHANGELOG.md`, with one pre-seeded stub section per phase (`## [Phase 1] - TBD`, etc.). When you finish a phase, do **not** edit `CHANGELOG.md` directly. Instead, output a draft changelog entry — a short "Added" list in the same style as the existing `[Baseline]` entry — for the human to review and paste in themselves. Along with it, explicitly flag whether `README.md`, `demo.gif`, or `intro-video-script.md` now describe the app inaccurately as a result of this phase's changes (e.g. Phase 3 adding Node/Python/Docker detection means the README's ecosystem list and the demo GIF's narrated flow are both out of date) — this check is a required part of every phase's output, not optional cleanup.

---

## Phase 1 — Quick wins (no new dependencies, no new API calls, ship these first)

**1.1 Collapse/Expand All Directories**
Add a small toggle button at the top of the file tree that expands or collapses every folder node at once. Pure DOM/state operation on the existing tree render — no new data needed.

**1.2 File-Type Icons in the Explorer**
Replace the generic file icon with an extension-to-glyph lookup table (JS/TS, JSON, Markdown, Apex, XML, Python, YAML, etc.). Do **not** pull in an icon font or SVG icon library — this should be a plain mapping object, zero new dependencies.

**1.3 Copy Raw File Content Button**
Add a "Copy Raw" button in the file preview header, using the exact same button/toast pattern as the existing "Copy Command" buttons. Copies the full raw content of the currently previewed file to the clipboard.

**1.4 Repo Health At-a-Glance Panel**
Add a small strip near the ribbon showing: last commit date, whether `.github/workflows/` exists (derivable from the tree data you already fetched at ingestion — no new API call needed for this part), license file presence/type, and open issue count. Only the last-commit-date and issue-count need a new API call each — keep it to 1–2 lightweight metadata calls total, not a full content fetch.

**1.5 Repo Architecture / Language Breakdown Bar**
A small horizontal progress bar showing file-type distribution (e.g. 60% Apex, 20% JSON, 20% Markdown), computed entirely client-side from the file tree you already have in memory post-ingestion. Zero new API calls — this is a pure computation over existing data.

---

## Phase 2 — Self-contained UX upgrades (one bundled dependency at most, still no new API calls)

**2.1 Line Numbers + Basic Syntax Highlighting**
Integrate a lightweight highlighter (Prism.js or Highlight.js core, plus only the languages actually needed: JS, JSON, Apex/Java-adjacent, XML, Markdown, Python, YAML). Download and bundle the minified build locally in the repo — see the hard constraint above, this must not be CDN-loaded. Line numbers can be done with a plain CSS counter on top of the existing `<pre><code>` block; no library needed for that part.

**2.2 In-File Search (Cmd+F inside the preview pane)**
Let the user search and highlight matches within the currently open file preview. Scope this to the active file only — do not conflate this with cross-file/repo-wide search, which is Phase 4.

**2.3 Deep-Link to Specific Lines**
Extend the existing "Copy Deep-Link" feature (already tested and working — `?repo=owner/repo`) to support a line-range fragment, e.g. `?repo=user/repo&path=skills/main.json#L12-L25`, which scrolls to and highlights that range on load. Build directly on the proven deep-link mechanism rather than a new one.

---

## Phase 3 — Ecosystem & validation expansion (extends the existing detection engine)

**3.1 Expanded Ecosystem Detection: Node/NPM**
Detect `package.json`. Do not hardcode `npm start`/`npm run build`/`npm test` as the only options — parse the actual `scripts` object and surface whatever script names the repo actually defines (many real projects use `dev`, `test:watch`, etc. instead of the conventional names).

**3.2 Expanded Ecosystem Detection: Python**
Detect `requirements.txt` and `pyproject.toml` as **separate** cases, not one generic "Python detected" card — they imply different install commands (`pip install -r requirements.txt` + venv setup vs. `poetry install` / `pdm install`).

**3.3 Expanded Ecosystem Detection: Docker**
Detect `Dockerfile` and/or `docker-compose.yml`, output the corresponding `docker build` / `docker compose up` command.

**3.4 Manifest Validation Warnings**
For Salesforce/Agentforce repos specifically, run a client-side check on the relevant manifest (`sfdx-project.json`, `skill.json`) for missing required keys or malformed JSON, and surface a warning before the user copies a CLI deploy command that would fail. This extends the signature-matrix parsing logic that already exists for ecosystem detection — same code path, one step further.

---

## Phase 4 — Higher-effort, new subsystems (design carefully, higher regression risk, review before merging)

**4.1 "Verify Before You Run" — pre-execution supply-chain lint**
Before displaying the install/deploy command, run a client-side heuristic scan of the relevant manifest for risky patterns: `postinstall`/`preinstall` scripts in `package.json`, `curl | bash` or `wget | sh` patterns in `Dockerfile`s, dependencies pointed at non-standard/unofficial registries, unpinned version ranges on scripts that run automatically. Surface an inline warning badge on the command card if something matches — do not block the copy action, just flag it. This is new pattern-matching logic across multiple manifest types; expect false positives during initial rollout and plan for a way to refine the ruleset over time.

**4.2 Scoped in-session content search**
Do **not** build unscoped "search every file in the repo" — on a repo the size of `forcedotcom/sf-skills` (4,000+ files), that would require fetching content for nearly every file in one action, which exhausts the unauthenticated 60/hr rate limit in a single search. Instead: build an incrementally-growing client-side index of only the files the user has already opened/previewed during the current session, and search within that. This stays consistent with the app's "optional token, works fine without one" architecture instead of quietly forcing a token requirement.

---

## Explicitly deferred — do not build in this roadmap without a separate decision

**Multi-tab preview canvas.** This isn't a small addition — it changes what the right-hand command-card panel means the moment 2+ files are open (whose install/deploy command shows?), which means re-touching the panel layout that was just simplified in the last redesign pass. Treat this as its own future UI-redesign project, not a Phase 4 item, if it gets picked up at all.

**Full unscoped global repo content search.** As covered in 4.2 — the only way to make this fast (GitHub's Code Search API) requires a token as a hard requirement, not an optional one, which is a real product-positioning decision, not just an engineering task. Don't build this version without that conversation happening first.
