# agentbrowse — command reference

Invoke any command with `npx agentbrowse <command>` (or `agentbrowse <command>` if installed). A global `--session <id>` flag (default `default`) selects an isolated browser + cookie jar; add `--json` to most commands for machine-readable output.

## Navigation & reading

| Command | What it does |
|---|---|
| `open <url>` | Navigate the session's browser to a URL. |
| `read [url]` | Read the current page (or open `<url>` first) as clean, token-bounded markdown. `--max-chars <n>` (default 8000), `--page <n>` for the next chunk when truncated. |
| `links [url]` | List navigable links on the current page, numbered for reuse by `click`. `--filter <text>` to narrow. |
| `snapshot [url]` | List actionable elements from the accessibility tree as `[ref] role "name" (state)`. `--filter <text>`, `--max <n>` (default 150). **Preferred way to act.** |
| `find <text...>` | Find elements by visible text; returns numbers reusable by `click`. |

## Data layer (capture & replay)

Modern sites fetch their content as JSON. These read and re-issue that data layer directly — structured, complete, and re-fetchable with your session's auth. (The JSON often carries more fields than the visible page, so it isn't always fewer tokens than `read`; reach for it when you need structured/complete data, pagination, or to skip rendering — and use `read` when you just want the visible text cheaply.)

| Command | What it does |
|---|---|
| `capture` | List the JSON/API calls the current page fetched, newest last, each with a stable `[id]`. Waits briefly for load-time fetches so it works right after `open`; `--wait` waits for a *new* call (e.g. after a click); `--new` shows only calls since the last navigation (this page's data layer). `--filter <text>`, `--method <m>`, `--max <n>`, `--json`. |
| `capture <id>` | Show one call's request + response body, token-bounded like `read` (`--max-chars`, `--page`, `--json` for parsed JSON). |
| `replay <id>` | Re-issue a captured call inside the session's auth, **without rendering the page**, for fresh data. `--query key=value` (repeatable) overrides URL params — ideal for pagination/filtering. `--json`, `--max-chars`, `--page`. |

Capture ids are monotonic and survive navigation, so `capture 8` → `replay 8` stays valid across page loads. Unknown ids exit `4` (target not found).

## Interaction

| Command | What it does |
|---|---|
| `click <target...>` | Click by snapshot ref (`click 3`), visible text (`click "Sign in"`), a number from `links`/`find`, or a CSS selector (`click "button.primary"`). |
| `type <selector> <text...>` | Type into a field by snapshot ref, field name, or CSS selector. |
| `fill -f name=value [-f …]` | Fill one or more form fields by name. Repeatable. |
| `submit [form]` | Submit the current form, or a specific one by selector. |

## Sessions & auth

| Command | What it does |
|---|---|
| `login <url> [--until <text>]` | Open a real (headed) browser to log in once; persists the session for later headless reuse. With `--until <text>` it auto-finishes when the URL contains that text; otherwise it waits for Enter. |
| `session save` | Persist the running session's cookies/localStorage. |
| `session load` | Reload saved state into the session. |
| `session clear` | Delete saved state for the session. |
| `stop` | Stop the session's background browser daemon (also self-stops when idle). |

## Distribution

| Command | What it does |
|---|---|
| `skill [target]` | Install this skill so coding agents use agentbrowse by default. `target` = `claude` \| `codex` \| `cursor` \| `gemini` \| `windsurf` \| `agents` \| `all` (default: auto-detect the agents configured in the current project). `--global` installs at user level, `--print` prints instead of writing, `--force` overwrites existing files. |

## Site manifests

If a `site.agent.json` manifest exists for the target site, run its named multi-step flows:

```bash
npx agentbrowse --site ./site.agent.json <command> [args...]
```

Each manifest command runs a predefined flow (e.g. open → type → submit → read) and returns the final read output. Inspect the manifest's `commands` for what's available.

## Exit codes

`0` ok · `1` internal · `2` usage (incl. `blocked_url`) · `3` navigation · `4` target not found (re-run `snapshot`/`links` for fresh refs) · `5` daemon. Errors are written to **stderr** as `{ "error": { code, message } }`.

**Navigation guard:** only `http(s)` schemes are allowed; local/private hosts (`localhost`, `127.0.0.1`, `10/172.16/192.168`) are blocked by default → `blocked_url` (exit 2). Hostnames are **DNS-resolved** and checked by their actual IP (so rebinding names like `127.0.0.1.nip.io` are caught), and redirects/subresources are re-checked. **Link-local/cloud-metadata (`169.254.x`, `metadata.google.internal`) stays blocked even with `WEBCLI_ALLOW_LOCAL=1`** — that flag only permits loopback/private for local dev. Saved session state is written `0600`.
