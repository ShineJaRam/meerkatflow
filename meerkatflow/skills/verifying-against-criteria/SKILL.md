---
name: verifying-against-criteria
description: Use when verifying a completed implementation against a spec/plan criteria file, after meerkatflow:execute finishes, before claiming work is done, when asked to review changed code against an md file, to continue a prior verification, to enforce a review score, or to summarize verification feedback. Verifier stance only — does not implement.
---

# Verifying Against Criteria

Verify an implementation against a markdown criteria file (a meerkatflow spec or plan) and leave durable, scored feedback.

**Core principle:** verifier, NOT implementer. Findings lead; never fix code unless the user explicitly switches the task to implementation.

## Locations

- Criteria file: meerkatflow plan/spec, normally `<docs>/plans/` or `<docs>/specs/`.
- State: `.claude/verify/state.json`
- Feedback: `.claude/verify/<criteria-basename>-feedback.md`

Create `.claude/verify/` if missing. It must stay out of git: if `git check-ignore -q .claude/verify` fails, add `.claude/verify/` to `$(git rev-parse --git-path info/exclude)`.

## State and locking

```json
{
  "status": "idle | blocked",
  "criteriaPath": "relative/path/to/criteria.md",
  "latestScore": 0,
  "feedbackCount": 0
}
```

- No state file → `idle`.
- `blocked` with `latestScore` < 90 → **reject any other criteria file.** Tell the user the previous verification is incomplete and name the locked path. Another pass on the **same** file is allowed and expected. If the user drops that work, set the state to `idle` and note it in the feedback file.

## Feedback file naming

For `plan.md`, in `.claude/verify/`:
- First pass `plan-feedback.md`; then `plan-feedback-1.md`, `plan-feedback-2.md`, …

Pick the next suffix from existing files + state. Never overwrite feedback/summary files unless the user asks.

## Review procedure

1. Read the criteria file; extract concrete criteria. Prefer an explicit section whose heading contains `Acceptance Criteria`, else derive them.
2. Change scope: if the caller gave a diff file (e.g. `final-diff.md`), use it. Otherwise `git diff --stat` then `git diff`, plus untracked files from `git status --short`.
3. Inspect changed files AND related paths: callers, shared state, schemas, docs.
4. Verify claims against actual code, not the criteria doc.
5. Hunt regressions, missing edge cases, integration mismatches, type errors, state bugs, stale assumptions. An edge case counts only when the state actually occurs (spec decisions, measurements in the project's docs, real data); type declarations alone do not settle it either way. Exception: input validation at trust boundaries, security and data-loss handling count whether or not the state has been seen.
6. Honor constraints passed by the caller (e.g. no deduction for missing tests when the constraints forbid adding test files).
7. Mark criteria you cannot judge here as **pending**: not measurable in this environment (no access, no data), or waiting on a user decision. Give the reason and what is needed. "Did not try" is not a reason. Pending criteria are not deducted; tell the user about each one. A criterion the user drops is written into the criteria file as a user decision before the next pass.
8. Score out of 100 on what code and measurements can decide. Write feedback with `file:line` references.

Prefer the project's validation commands; no costly or destructive commands without reason.

## Scoring

- **90–100**: meets criteria, no material bugs. Minor polish may remain.
- **80–89**: mostly correct, ≥1 meaningful bug, missing edge case, integration risk or requirement gap.
- **60–79**: partial; multiple important issues.
- **<60**: major failures, broken behavior, or too little to validate.

**< 90:** state `blocked` (path, score, feedbackCount); tell the user verification continues on this target until ≥ 90.
**≥ 90:** state `idle` (path, score, feedbackCount).

## Feedback format

Headings and body in the user's language; keep this structure and order.

```markdown
# <criteria basename> Feedback

## Score
<score>/100

## Verdict
<pass/fail summary and whether verification is complete>

## Findings

### 1. <title>
- Severity: High | Medium | Low
- Location: `path/to/file.ts:123`
- Problem: <what is wrong>
- Impact: <why it matters>
- Recommendation: <concrete next action>

## Criteria check
- [x] <criterion met>
- [ ] <criterion not met + reason>
- [~] <criterion pending + reason and what is needed>

## Notes
<commands run, what was not run, assumptions, residual risk>
```

No findings → say so explicitly and note residual risk.

## Summary

Only when the user asks: read all feedback files for that criteria file and write `<criteria-basename>-feedback-summary.md` (outcome, recurring issues, resolution, latest score, residual risks).
