# AGENTS.md

## Cursor Cloud specific instructions

This repo is a single-file, fully static web app (`index.html`) — vanilla HTML/CSS/JS
with IndexedDB for persistence. There is **no build step, no package manager, and no
dependencies** to install (no `package.json`, no lockfile). There are also no automated
tests, linters, or CI build scripts in the repo.

- **Run (dev):** serve the repo root over HTTP and open `index.html`, e.g.
  `python3 -m http.server 8000` then visit `http://localhost:8000/index.html`.
  Prefer serving over HTTP rather than opening via `file://` so IndexedDB and the
  bundled Phosphor fonts (`fonts/`) behave consistently.
- **Develop:** edit `index.html` and refresh the browser (see README "Development").
- **Data persistence:** state lives in the browser's IndexedDB (stores: `companies`,
  `applications`, `weeklyTasks`). Data persists per-browser-profile across refreshes;
  clear IndexedDB to reset to an empty state.
- **Lint/test/build:** none configured. If asked to lint/test, note there is no tooling
  in-repo unless it is added.
- **External resources:** the page also references the Inter font via Google Fonts CDN;
  the core app still works offline/without it (falls back to system fonts).
