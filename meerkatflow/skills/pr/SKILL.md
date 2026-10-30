---
name: pr
description: Use when the user asks for a PR body, PR title or commit message draft after work is done. Produces drafts only; never commits, pushes or opens the PR.
---

# PR / commit draft

Never run `git commit`, `git push` or `gh pr create`. Output drafts only, unless the user explicitly says so in this conversation.

Drafts follow the repo's language (what recent commits and PRs use), not the chat language.

## Source of truth

Read the actual diff first (`git diff <base>...HEAD`, or the plan workspace's `final-diff.md`). Describe what the code does, not what the plan intended. Base follows `meerkatflow:review` §1: on a branch cut from another unmerged branch, describe only this branch's own work.

## Commit message / PR title

Order of precedence:
1. A format the user or project rules state (session rules, CLAUDE.md, CONTRIBUTING).
2. The repo's recent history: `git log --no-merges -10 --format=%s`. Copy its prefix convention, language and length.
3. No convention visible → one line, `<type>: <what changed>` (`feat`, `fix`, `refactor`, `docs`, `chore`).

## PR body

Order of precedence:
1. A template the user or project rules state.
2. The repo's `.github/pull_request_template.md` (or `PULL_REQUEST_TEMPLATE/`).
3. Default:

```markdown
## Summary

<One or two sentences. What changed and why.>

## Background

<The problem. What the code was doing and why that was bad.>

## Changes

- **<One-line summary.>** <One or two lines of detail if needed: behavior the user or caller notices. No function, file or variable names unless the change is about them.>
```

- Scope limited on purpose (one tab, one screen, one entry point) → say so in the body.
- Measurement tables, review points, verification logs: only when asked. Exception: a change made to move a number (performance, bundle size) carries the before → after figures you measured.
- Numbers you did not measure yourself: state the source in one line outside the body.
- End with the attribution footer the harness provides.
