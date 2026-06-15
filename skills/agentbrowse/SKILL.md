---
name: agentbrowse
description: Drive a real web browser from the terminal with `npx agentbrowse` to navigate, read, and interact with live websites. Use whenever a task needs a real or interactive page rather than a static fetch, such as opening a URL, clicking links or buttons, filling and submitting forms, reading JavaScript-rendered or paginated content as clean markdown, following multi-step flows, or reusing a logged-in session.
---

# Driving websites with agentbrowse

`agentbrowse` is a CLI that drives a real browser (Playwright) from the shell. Reach for it instead of guessing URLs, asking the user to click things, or giving up on pages that need JavaScript or a login.

## Run it

No install step — invoke with `npx agentbrowse <command>`. The first browser command auto-starts a background browser that persists across commands. If you ever get a "browser not installed" error, run `npx playwright install chromium` once and retry.

## The session model

One persistent browser holds state between commands. `open` once, then `read` / `snapshot` / `click` / `fill` act on that same live page — cookies, scroll position, and DOM are all preserved. Use `--session <id>` to keep independent parallel sessions (e.g. one per site or task). `npx agentbrowse stop` frees the browser; it also self-stops when idle.

## The core loop

```bash
npx agentbrowse open https://news.ycombinator.com   # navigate
npx agentbrowse snapshot                             # list actionable elements with [refs]
npx agentbrowse click 12                             # act by ref
npx agentbrowse read                                 # clean, token-bounded markdown
```

## Rules that make it reliable

- **Act via `snapshot`.** It returns every actionable element as `[ref] role "name" (state)` — e.g. `[12] link "Your EPUB Is Fine"`. Then `click <ref>` or `type <ref> "text"`. Refs resolve by role+name, so they survive CSS/DOM changes — far more robust than selectors. If a ref goes stale the tool returns a **fresh snapshot inside the error** (`stale_ref`, exit 4) — just re-pick from it.
- **Targeting priority for `click`/`type`:** (1) a `snapshot` ref → `click 3`; (2) visible text → `click "Sign in"`; (3) a number from the last `links`/`find`; (4) a CSS selector → `click "button.primary"`.
- **Reading:** `read` is token-bounded (`--max-chars`, default 8000). If it reports more pages, request `--page 2`, etc. Add `--json` to parse `{ title, markdown, page, totalPages, state }`. Every text output ends with a `url | title | links` footer so you always know where you are.
- **Forms:** `fill -f email=me@x.com -f password=…` then `submit`, or `type <field> "text"` for a single field.
- **Errors go to stderr** as `{ "error": { code, message } }`. Exit codes: `4` = target not found (re-run `snapshot`/`links` for fresh refs), `3` = navigation, `2` = usage, `5` = daemon.

## Authentication — never type passwords

If a page needs login, do **not** type credentials. Ask the user to run once:

```bash
npx agentbrowse login <url> --session <id>
```

A real browser opens for them to log in; the session is saved and you can then operate headlessly under that `--session`. Never put credentials in command arguments.

See [reference.md](reference.md) for the full command list and flags.
