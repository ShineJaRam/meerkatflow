---
name: review
description: Use when the user asks to review the current changes without a plan to grade against — "review the git changes as a senior developer", "is anything overbuilt?", "is this ready to commit / open a PR?". Reads the working diff for defects and overbuilding, runs static checks, says what was not measured and gives a ready / not ready verdict. Does not fix until told.
---

# Review (current diff, no plan)

Reviewer stance. Report first; fix only what the user then picks.
With a plan or spec to grade against, use `meerkatflow:verifying-against-criteria` instead.

## 1. Scope

- Diff = `git diff <main>...HEAD` + uncommitted changes + untracked files (`git status --short`). Say which in one line (N files, +a/−b).
- Base: if the branch was cut from another unmerged branch (reflog `branch: Created from <x>`, or the merge-base with a feature branch is newer than with main), diff against that branch, not main, so a parent branch's work is not reviewed as this one's. The first line names the base. Unsure → ask.
- Read the whole diff yourself, in this session.

## 2. Checks

- Run the project's typecheck and lint on the changed files, and the tests that cover them. Quote the result. Failures that also exist at HEAD are not this diff's.
- Constraint scan: for every standing rule that can be checked mechanically (comment policy, test-file policy, formatting), scan the added lines.

## 3. Read for

- **Defects:** wrong behavior, regressions against the previous code (a fallback or guard that disappeared), swallowed errors, races, edge cases. Check callers when a contract or shared state changed.
- **Data and contracts** (when the diff touches storage or an interface others call): migrations with no way back, changes that break existing rows or old clients, multi-step writes without a transaction, unbounded or N+1 queries, new entry points without authorization, secrets or personal data in logs.
- **Overbuilding:** lines the request did not need, an abstraction with one user, a helper the platform or an existing util replaces, a new helper that only wraps a short expression an inline form would cover, changes outside the request's scope. Name what to delete.
- **Consistency:** naming, existing patterns, shared components the neighbours use.

Every finding: file:line, what happens to whom, the fix, and how likely it is. Trace the path before rating likelihood; never rate by how the code looks.

Before calling a guard dead or asking for a new one, find out whether the state actually occurs: grep the project's docs (spec decisions, investigation notes, measurements) and look at real data or responses. Type declarations are not evidence of what arrives at runtime. A finding that contradicts a recorded user decision is dropped. Exception: input validation at trust boundaries, security and data-loss handling count whether or not the state has been seen.

## 4. Report

1. Verdict first: ready / ready after N fixes / not ready.
2. Numbered findings, most severe first, each marked `must` / `should` / `optional`, so the user can answer "apply 1 and 3".
3. Checks run and their results.
4. Not measured: what was not checked in the running app. If the diff changes what the user sees and nothing was measured, say the verdict is static-only.

## 5. After the user picks

Apply only the picked items, rerun the checks. If the diff changes what the user sees or what a caller gets, run `meerkatflow:measure` on the touched paths before reporting, without being asked; say what could not be measured. Drafts → `meerkatflow:pr`.
