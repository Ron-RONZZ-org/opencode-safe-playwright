# opencode-safe-playwright

Safe browser automation for [opencode](https://opencode.ai) with **per-session Chromium isolation**.

Each opencode session gets its **own Chromium process** and browser profile directory.
Session A's browser operations never interfere with Session B's.

## Features

- **Per-session isolation** — `Map<sessionID, SessionState>` instead of shared module state
- **Safety hooks** — auto-kills zombie Chromium processes, auto-clears stale lock files before every operation
- **Diagnostic tools** — `browser_health` and `browser_clean` for agents
- **Guidance injection** — `<BROWSER_SAFETY>` instructions auto-injected into every LLM turn
- **Per-session profile cleanup** — profile directories removed when sessions end
- **Idle watchdog** — automatically closes browser after 30 minutes of inactivity

## Tools

| Tool | Purpose |
|------|---------|
| `browser` | Main browser control (start, stop, open, navigate, snapshot, screenshot, click, type, evaluate, wait, close, back) |
| `browser_start` | Quick start in headed (visible) or headless mode |
| `browser_snapshot` | Take ARIA snapshot with interactive element refs |
| `browser_click` | Click element by snapshot ref |
| `browser_type` | Type text into element by snapshot ref |
| `browser_health` | Check Playwright/Chromium installation, profile state, running processes |
| `browser_clean` | Kill zombie processes, remove stale lock files. `force: true` destroys all session profiles |

## Usage

Add to your `opencode.jsonc`:

```json
{
  "plugin": ["opencode-safe-playwright"]
}
```

Or load from a local path:

```json
{
  "plugin": ["./path/to/opencode-safe-playwright/src/index.ts"]
}
```

## How it works

```
opencode serve
└── plugin state: Map<sessionID, SessionState>
    ├── ses_A → Chromium process A → profile_A
    └── ses_B → Chromium process B → profile_B
```

Profile directories: `~/.opencode/browser-profile/sessions/<sanitized_session_id>/`

Each session's browser context is launched via `chromium.launchPersistentContext()`
with its own user data directory. When a session ends (`session.idle` / `session.deleted`),
the browser is closed and the profile directory is removed.

## Safety

The plugin automatically:

1. **Before any browser action**: kills orphaned Chromium processes and removes stale lock files from the profile
2. **After any browser error**: kills orphaned processes
3. **On session end**: closes the browser and removes the profile directory
4. **On inactivity**: closes the browser after 30 minutes

## Guidance injection

The `<BROWSER_SAFETY>` XML block is injected into the first user message of every
LLM turn via `experimental.chat.messages.transform`, and re-injected during session
compaction via `experimental.session.compacting`. This ensures agents always see
the safety checklist.

## Development

```bash
git clone https://github.com/Rong-Zhou-FR/opencode-safe-playwright.git
cd opencode-safe-playwright
npm install
npm test
```

## License

MIT

## Credits

Inspired by the original [opencode-browser-plugin](https://github.com/heimoshuiyu/opencode-browser-plugin) by heimoshuiyu.
