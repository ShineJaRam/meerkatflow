---
name: measure
description: Use when checking something in the running system — a browser on the dev server, or a service through its API, database, logs or jobs — to reproduce a bug, confirm a fix, read requests or state, or as the toolkit behind meerkatflow:verify-runtime. Covers finding the right instance, driving a browser with whatever tool the session has, reading state, service checks, and keeping large results in files.
---

# Measure (check in the running system)

During `meerkatflow:execute`, run after implementers finish: while source is being edited, dev-server full-reloads wipe tab state.

**Large results go to files, not into the conversation.** Anything bigger than a screenful (a sweep over many nodes, routes or requests; response bodies; log excerpts):
- Shell-run checks: write output to `<docs>/measure/YYYY-MM-DD-<topic>.<ext>` and print only counts and the failing rows. Read details back with Grep or an offset.
- Browser tab: keep the full result on `window.__r` and return only the aggregate (counts, failing keys). Fetch single rows on demand.
- Cite the file path in the report.

Browser check → sections 1–4. Service check → section 5. Full stack: check the API first, then the screen; when the screen is wrong, the request and response at the boundary say which side.

If a personal or project plugin ships its own measure notes (server start, login, which browser tool, tool pitfalls, project components), load that skill too before starting. Its tool choice wins over section 2.

## 1. Find the right server

Ports are not fixed per worktree. Never trust a port written in a spec. List the listeners in the dev server's port range (`<from>`–`<to>`, e.g. vite 5173–5180) with their command lines.

Windows (`netstat`; `Get-NetTCPConnection` can miss a live listener):
```powershell
netstat -ano | Select-String 'LISTENING' | ForEach-Object { $f = -split $_.Line; $port = [int]($f[1] -split ':')[-1]
  if ($port -ge <from> -and $port -le <to>) { $p = Get-CimInstance Win32_Process -Filter "ProcessId=$($f[4])"; "$port  $($p.CommandLine)" } } | Sort-Object -Unique
```

macOS / Linux:
```bash
lsof -nP -iTCP:<from>-<to> -sTCP:LISTEN          # port and PID
ps -o command= -p <PID>; lsof -a -p <PID> -d cwd -Fn   # command line and cwd
```

Use the port whose command line or cwd contains the **current worktree path**. If none, start one (measure notes say how; otherwise the project's dev script) and tell the user which port you started.

## 2. Drive the browser

Use the browser automation tool this session has: a browser extension's MCP tools, a Playwright-style CLI or REPL, a headless runner. Check which exist before saying the browser cannot be checked. Rules that hold for any of them:

- **Right machine.** The browser must run on the machine serving `localhost`. A remote browser gets connection refused.
- **Rerunnable script.** A bug that is hard to trigger or will be reproduced again, and any check over several pages or states, goes in one script file `<docs>/measure/<topic>.<ext>` when the tool runs scripts: open the page → install console and fetch capture → the steps → print console errors, failed requests and the checked values → save a screenshot at each step that matters. The same file is the repro before the fix and the check after it. A tool without scripts: write the steps as a numbered list and follow them exactly both times.
- **A tool's own agent is not a measurement.** When a tool can run its own LLM agent ("click these and tell me"), its answer is an account. Confirm a claimed pass or fail by reading the value yourself.
- **Dialogs.** Some tools freeze on `confirm`/`alert`, others accept them silently, which lets a confirm-guarded write through. Ask before clicking anything that might confirm a write on shared data.
- **Background tabs.** Automated or background tabs often throttle `requestAnimationFrame`, timers and CSS transitions: transitions stall, `getComputedStyle` returns pre-transition values, lazily rendered lists and popovers may not exist yet. Disable transitions before reading styles. Never await a frame; use timers and short polling. Frame timing and scroll performance need a foreground tab: give the user a self-contained console snippet that prints JSON.
- **Window size.** If the tool cannot resize the viewport, force the element's width from JS.
- **Screenshots.** Save to a file and report the path; never paste images or long logs into the conversation. To judge visuals, open the file yourself. A blank or stale screenshot is a broken observation: force a render and retake before judging.
- Report whatever the tool could not observe as not measured, with the reason.

## 3. Reading state

- Request/response bodies: wrap `window.fetch` (and the HTTP client the app uses) once, guarded so a second install is a no-op, and store the calls you need on `window.__x`. Return only the needed fields.
- Component state: from one element inside the target, walk the framework's parent links upward (e.g. Vue `__vueParentComponent`, React fiber). Never scan `document.querySelectorAll('*')` on big pages.

## 4. When an observation looks wrong

Suspect the method before the program: a capture hook installed twice doubles counts, a background tab freezes renders, a cached response hides a change, a remote browser hits a different server. Before reporting a call count, confirm the hook was installed once and state the entry path (fresh load vs in-app navigation).

## 5. Service checks (API, database, jobs)

- **Right instance.** Same rule as ports: use the process whose cwd is the current worktree. State the base URL and database name you hit. Never a shared, staging or production instance unless the user named it.
- **Real entry point.** Send the HTTP request, run the CLI, enqueue the job. Calling the function directly is a test, not a measurement. Example: `curl -sS -o <file> -w '%{http_code} %{time_total}\n' ...`.
- **Effect, not only the response.** Read back what changed with a read-only query: rows written, event or queue message emitted, log line. Compare before and after.
- **Cases** (those the checklist line implies): success, invalid input, unauthorized, not found, the same request twice, two at once.
- **Writes.** Use data you create and remove it afterwards. Never change existing shared data. Migrations run only on a local, disposable database; if the project has down migrations, check up and down. Anything irreversible or outward-facing (shared database, mail, payment, third-party API) → ask first.
- **Secrets and personal data.** Never print tokens or connection strings. Report counts or masked values, not rows of personal data.

## 6. Report

- First line: the verdict (works / does not / partly, plus what was not measured). Then a table: condition → expected → observed → pass/fail.
- State coverage: which variants (tabs, item types, locales, kinds of data) you measured and which you did not. A measurement used to justify deleting or simplifying code must cover every variant; a variant with no data proves nothing.
- Only what no tool here can observe (frame timing in a throttled tab, permission-gated screens) goes to the user, with what to look at.
- A screen, element or call is `measured` only when you saw it in this run; otherwise mark it `code-only`.
- Checking a plan's runtime checklist or acceptance criteria → use `meerkatflow:verify-runtime`, which records the result.
