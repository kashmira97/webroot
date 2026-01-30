# AGENTS.md

This is the equivalent to `CLAUDE.md`.

Use modern, clean designs inspired by claude.ai.

Primary guidance files:
- `/localsite/AGENTS.md`
- `/team/AGENTS.md`

Submodule overview:
- `codechat/README.md`

Key standards (from linked AGENTS files):
- HTML: use `/localsite/start/template/index.html` for new pages; include `<meta charset="UTF-8">` except in redirects or template fragments.
- DOM waits: never use `setTimeout` for DOM; use `waitForElm(selector)` from `localsite/js/localsite.js` (confirm it is included first).
- Hash state: prefer `getHash`, `goHash`, `updateHash`, and `hashChangeEvent` from `localsite/js/localsite.js`.
- Paths: never hardcode user-specific paths; use relative paths or repo-root discovery.
- Git: only run push/pull via `./git.sh` and only commit/push when the user explicitly asks.
- **Push scope**: when user says "push [repo]", push ONLY that specific repository. Do not use `git add .` or stage unrelated changes. Examples:
  - "push localsite" → push only localsite submodule changes
  - "push team" → push only team submodule changes
  - "push" or "push all" → push webroot + all submodules via `./git.sh push`

Claude Code sessions:
- Session history: `~/.claude/history.jsonl` (JSONL format with sessionId, timestamp, display, project)
- Use Python or `jq` to parse efficiently; avoid multiple `awk` attempts on macOS

Start commands:
- `start server` — starts Python HTTP server without Flask (`desktop/install/quickstart.sh`)
- `start rust` — Rust API server (from `team` repo)
- `start flask` — starts both `cloud` and `pipeline`
- `start cloud` — Flask for `cloud/run` (RealityStream), local + deploy to Google Cloud
- `start pipeline` — Flask for `data-pipeline/admin`
- `start html` — bare bones without Python (not needed if you ran `start server`)

Ports:
- `8887` — Python HTTP server (`desktop/install/quickstart.sh`)
- `8081` — Rust API server (from `team` repo)
- `5001` — Data-Pipeline Flask server
- `8100` — Cloud/run Flask server
