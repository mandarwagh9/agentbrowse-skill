---
name: agentbrowse
description: Drive a real web browser from the terminal with `npx agentbrowse` to navigate, read, and interact with live websites. Use whenever a task needs a real or interactive page rather than a static fetch, such as opening a URL, clicking links or buttons, filling and submitting forms, reading JavaScript-rendered or paginated content as clean markdown, following multi-step flows, or reusing a logged-in session. It can also capture the JSON APIs a page fetches and replay them with your saved login, so you read structured data instead of scraping rendered HTML.
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

## Read the data layer instead of scraping

Modern sites fetch their content as JSON. That data layer is the complete, structured source the page is built from — reach for it when you need data programmatically, want fields the page doesn't show, need to paginate/filter, or want to re-fetch with your login without re-rendering. (It's not necessarily *fewer tokens* than `read` — APIs often return more fields than the visible page — so when you just want the visible text cheaply, use `read`.)

```bash
npx agentbrowse open https://example.com/catalog
npx agentbrowse capture                  # list the JSON/API calls the page made, with [ids]
npx agentbrowse capture 8                # the full JSON body of one call
npx agentbrowse replay 8 --query page=2  # re-issue it (with your saved login) for fresh data
```

- `capture` waits briefly for load-time fetches, so running it right after `open` works. After a click that triggers a fetch, use `capture --wait`. After navigating around, `capture --new` shows only the current page's calls.
- `capture <id>` is token-bounded like `read` (`--max-chars`, `--page`, `--json` for parsed JSON).
- `replay <id>` re-issues the captured request inside your session (cookies/auth apply) **without re-rendering the page** — ideal for paging/filtering via repeatable `--query key=value`. Prefer it over another `open` + `read` when you just need more data. Capture ids survive navigation, so `capture 8` → `replay 8` stays valid.

## Rules that make it reliable

- **Act via `snapshot`.** It returns every actionable element as `[ref] role "name" (state)` — e.g. `[12] link "Your EPUB Is Fine"`. Then `click <ref>` or `type <ref> "text"`. Refs resolve by role+name, so they survive CSS/DOM changes — far more robust than selectors. Refs belong to the page you snapshotted: once the page navigates, acting on an old ref fails with `stale_ref` (exit 4) and the error's `data.elements` shows the current page. **Run `snapshot` again to get live refs, then act** — don't blind-retry the same number.
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
