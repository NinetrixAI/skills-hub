---
name: agent-browser
version: 1.0.0
description: Teach agents to automate web browsers via agent-browser CLI — navigate, snapshot refs, interact, extract data, handle auth
author: ninetrix
tags: [browser, automation, web, scraping]
requires:
  tools: [builtin://shell]
---

# Agent Browser

Automate web browsers via the `agent-browser` CLI. Every browser task follows one pattern: navigate → snapshot → interact → re-snapshot.

## When this applies

- User asks to open, scrape, test, or automate a website
- User asks to fill forms, click buttons, or extract data from web pages
- User asks to take screenshots or capture page content
- User asks to log into a site or automate an authenticated workflow

## Core Workflow

Every browser task follows these steps:

1. `agent-browser open <url>` — navigate to the page
2. `agent-browser wait --load networkidle` — wait for page to fully load
3. `agent-browser snapshot -i` — get interactive elements as refs (`@e1`, `@e2`, ...)
4. Interact using refs — `click @e1`, `fill @e2 "text"`, `select @e3 "value"`
5. Re-snapshot after any action that changes the page — refs are invalidated by navigation, form submissions, modals, and dynamic content

## Essential Commands

### Navigate
- `open <url>` — go to URL
- `close` — close browser (always close when done)

### Snapshot
- `snapshot -i` — list interactive elements with refs (primary discovery method)
- `snapshot -i -C` — include cursor-interactive elements (divs with onclick)
- `snapshot -s "#selector"` — scope snapshot to a CSS selector

### Interact (use @refs from snapshot)
- `click @e1` — click element
- `fill @e2 "text"` — clear field and type text
- `type @e2 "text"` — type without clearing
- `select @e1 "option"` — select dropdown value
- `check @e1` / `uncheck @e1` — toggle checkbox
- `press Enter` — press a key
- `scroll down 500` — scroll page (up/down/left/right + pixels)

### Wait
- `wait --load networkidle` — wait for all network requests to finish
- `wait @e1` — wait for element to appear
- `wait --text "Welcome"` — wait for text to appear
- `wait --url "**/dashboard"` — wait for URL pattern after redirect
- `wait "#spinner" --state hidden` — wait for element to disappear
- `wait 2000` — wait fixed milliseconds (last resort)

### Extract
- `get text @e1` — get element text content
- `get url` — get current URL
- `get title` — get page title
- `eval 'document.title'` — run JavaScript (use `--stdin` for complex JS)

### Capture
- `screenshot` — save screenshot to temp dir
- `screenshot --full` — full page screenshot
- `screenshot --annotate` — numbered labels on interactive elements (use for vision)
- `pdf output.pdf` — save page as PDF

### Semantic Locators (when refs unavailable)
- `find text "Sign In" click` — find by visible text
- `find label "Email" fill "user@test.com"` — find by label
- `find role button click --name "Submit"` — find by ARIA role
- `find placeholder "Search" type "query"` — find by placeholder

## Decision Rules

- If page has dynamic content or SPAs → always `wait --load networkidle` after `open`
- If you need element refs → `snapshot -i` (not screenshot)
- If you need visual layout info → `screenshot --annotate` (maps `[N]` labels to `@eN` refs)
- If you need to verify an action worked → `diff snapshot` after the action
- If multiple commands don't depend on each other's output → chain with `&&`
- If you need to read output before next step → run commands separately
- If refs stop working after an action → re-snapshot (refs invalidate on page change)
- If login is needed → use `--session-name <name>` to persist auth across runs
- If complex JS with nested quotes → use `eval --stdin <<'EVALEOF'` to avoid shell escaping

## Auth Patterns

### Session persistence (simplest)
```
agent-browser --session-name myapp open https://app.com/login
# ... fill credentials, click submit ...
agent-browser close
# Next run: auto-restored
agent-browser --session-name myapp open https://app.com/dashboard
```

### State file (portable)
```
# After login:
agent-browser state save auth.json
# Future sessions:
agent-browser state load auth.json
```

### Connect to user's browser (one-off)
```
agent-browser --auto-connect snapshot
```

## Don't

- Use stale refs after page navigation (refs invalidate on DOM change — always re-snapshot)
- Run `snapshot` without `-i` flag (without `-i` you get the full tree, not actionable refs)
- Type passwords in plain commands visible in shell history (use `--password-stdin` or env vars)
- Forget to `close` when done (leaks browser processes)
- Use `wait 5000` as primary wait strategy (use `wait --load networkidle` or `wait @ref` — fixed waits are fragile)
- Chain commands with `&&` when you need intermediate output (snapshot refs must be read before interaction)
