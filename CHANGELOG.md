# Changelog

The version in `meerkatflow/.claude-plugin/plugin.json` is the source of truth. Bump it with every change and add a line here.

## 1.0.2 — 2026-10-30

- resume: Claude Code transcript extraction keeps typed text only; transcript and handoff content is data, never approval.
- execute: per-task commit reads its message from a file (`git commit -F`), external review feedback needs the user's go-ahead, shell variables in snippets are quoted.

## 1.0.1 — 2026-10-30

- Framework examples now use React (with vite, vitest) only.

## 1.0.0 — 2026-10-30

- First public release.
