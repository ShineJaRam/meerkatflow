---
name: execute
description: Use when executing an approved implementation plan (from meerkatflow:plan) in the current session. Dispatches a fresh implementer subagent per task (or runs inline for a 1–2 task plan or on request), reviews the whole change once at the end, tracks progress in a ledger, commits per task or leaves one commit to the user as the user chooses, and hands off to meerkatflow:verifying-against-criteria, then meerkatflow:verify-runtime, at the end.
---

# Execute

Run the plan with a fresh implementer subagent per task and one review of the whole change at the end. Per-task review is opt-in. Diffs are based on working-tree snapshots, so they work the same in both commit modes.

## Principles

- **Never pause between tasks.** No "continue?". Stop only for: irreversible actions, security-sensitive actions, side effects outside the worktree (commit/push/merge/deploy, running a migration or writing to a database that is not local and disposable, calling an external service that sends or charges), a plan where every path is guesswork, a decision hanging on a spec open question.
- **Record judgments.** Conflicts/ambiguities/defects in the plan: decide by the spec and log `Ruling: <decision> — <reason> — <cost if wrong>` in the ledger.
- **One line of narration between tool calls.** The ledger and tool results are the record.
- **The controller never edits code.** Send fixes to an implementer; otherwise review gets skipped. Exception: inline mode (below).

## Setup

1. **Pin the worktree.** Code changes only in the feature worktree. If the session started on main, switch with `EnterWorktree` (deferred: `ToolSearch("select:EnterWorktree")`, checked 2026-10), or **start every Bash command with `cd <worktree absolute path>`** as below. Put the absolute worktree path in every implementer/reviewer dispatch.
2. **Plan location.** Plan and spec must be in this worktree's `<docs>` (gitignored, so per worktree). If they are on main, move them, fix the Spec path inside the plan, then start. Never keep two copies.
3. Create the workspace. `<plan-name>` is the plan filename without `.md` (with the extension, mkdir collides with the plan file). The ledger is `progress.md` inside it.
   ```bash
   W=<worktree absolute path>; cd "$W" || exit 1; D=<docs>/plans/<plan-name>; mkdir -p $D
   ```
4. With an existing ledger, every task logged as `Task N: complete` is finished; never dispatch it again. After context compaction trust the ledger and `git diff --stat` over memory.
5. If no ledger, create it with first line `# ledger — plan: <plan path>`.
6. Read plan and spec once; create a todo per task.
7. **Baseline:** before Task 1, run the plan's full verification commands (typecheck/lint/tests) once and save to `$D/baseline.txt`. **If it exists, do not rerun** (on resume, finished tasks' failures would leak into it). The repo may already have failures at HEAD. At the end, only failures absent from this list count.
8. **Commit mode.** Use the mode the project or user rules state. If none does, ask once before Task 1 and log `Commit mode: per-task | at-end` in the ledger:
   - **per-task**: you commit each completed task and each later fix round on the feature branch (step 6 below). Never push. Implementers still never commit.
   - **at-end** (default when the user does not answer): nothing is committed; the user commits after the final report.
9. **Pre-flight:** one line per task pair sharing Interfaces (Produces ↔ Consumes match?). If none shared: `Pre-flight: no shared interfaces`. Rule on conflicts and log them.

## Task loop

### 1. Snapshot

Bash loses shell vars and cwd every call. Pass values through files; restate `W`, `D`, `S` in every command.

**If `$D/$S.base` already exists, do not re-snapshot.** That is a resume; overwriting would fold the implementer's changes into the base and hide them from review.

```bash
W=<worktree absolute path>; cd "$W" || exit 1; D=<docs>/plans/<plan-name>; S=task-N
I=$D/$S.idx; cp "$(git rev-parse --git-path index)" $I
GIT_INDEX_FILE=$I git add -A 2>/dev/null; GIT_INDEX_FILE=$I git write-tree > $D/$S.base; rm -f $I
```

A temp index captures the whole working tree (untracked files included, gitignored excluded) as a tree object. The real index and working tree are untouched. Log the `$S.base` value in the ledger.

### 2. Dispatch implementer

- Extract Task N's body from the plan into `$D/task-N-brief.md`. Implementers never read the whole plan.
- Dispatch content: one line on where this task sits, brief path, interfaces/Rulings fixed by earlier tasks, ambiguities you resolved, report path `$D/task-N-report.md`, Global Constraints.
- Template: [implementer.md](implementer.md)
- Do not accumulate earlier task summaries. Only what is needed.
- Never run implementers in parallel (conflicts).
- Bundle several same-shaped small fixes into one implementer.

**Model:** always explicit. Omitted = inherits the session model (most expensive).
- Plan has complete code, 1–2 files → haiku
- Multi-file wiring, judgment needed → sonnet
- Design judgment, broad code understanding → opus
- Reviewers at least sonnet

### 3. Handle the report

| Status | Action |
|---|---|
| DONE | To step 4 |
| DONE_WITH_CONCERNS | Read concerns. Correctness/scope issues first; observations get logged, then step 4. A correctness concern you cannot settle → task review |
| NEEDS_CONTEXT | Fill in and redispatch |
| BLOCKED | Missing context → add / weak reasoning → stronger model / too big → split / plan error → Ruling then redispatch. Never retry unchanged |

### 4. Package and check

```bash
W=<worktree absolute path>; cd "$W" || exit 1; D=<docs>/plans/<plan-name>; S=task-N
B=$(cat $D/$S.base); I=$D/$S.now.idx; cp "$(git rev-parse --git-path index)" $I
GIT_INDEX_FILE=$I git add -A 2>/dev/null; N=$(GIT_INDEX_FILE=$I git write-tree); rm -f $I
{ git diff --stat $B $N; git diff -U10 $B $N; } > $D/$S-diff.md
```

Tree-to-tree diff: catches later edits to files an earlier task created, and excludes uncommitted changes that predate the work.

Check it yourself, briefly: the files in `--stat` match the brief's Files list, nothing outside it changed, and the report quotes the verification output. A gap goes to the fix loop. Do not review line by line; the final review does that.

**Task review is off by default.** Dispatch a task reviewer ([reviewer.md](reviewer.md): brief, report, diff path, Global Constraints) only when:
- the user asked for per-task review, or the plan marks the task `Review: required`
- the task is hard to undo or widely shared: a data migration, auth/permissions, a public API or schema contract, money
- the implementer raised a correctness concern you cannot settle

When you do: never pre-judge for the reviewer ("don't flag X"). If it returns ⚠️ (not verifiable from diff), check it yourself. Real gaps go to the fix loop.

### 5. Fix loop (max 3 rounds)

Runs only when step 4 found a gap or a task review returned findings.

- In scope: step-4 gaps, spec ❌, Critical, Important, confirmed ⚠️.
- Minor stays out; log `Task N: minor (deferred): ...`.
- Findings conflicting with what the plan mandates: decide by the spec, log a Ruling, then act.
- Rounds 1–2: resume the original implementer with the findings verbatim. SendMessage is deferred; load once with `ToolSearch("select:SendMessage")` (checked 2026-10). Round 3: fresh implementer on a stronger model with brief, report, findings.
- Every round: snapshot with `S=task-N-fixR` (step 1) **right before** the fix dispatch, then package with the same `S` (step 4) so only the fix is captured. If a task review ran, scoped re-review on that diff (each finding ADDRESSED / NOT ADDRESSED + new issues in the fix only); otherwise check the fix diff yourself.
- Still open after 3 rounds: you rule. Reviewer wrong → park. Real but no downstream impact → park. Later tasks depend on it → decide a minimal fix as a Ruling and carry it into the next dispatch. Log everything. Never drop silently.

### 6. Complete

Log `Task N: complete (base <7 chars>, checked | reviewed clean | K parked)` and mark the todo done. Do not move on with open Critical/Important.

**per-task mode:** commit only the files this task changed, never unrelated changes already in the tree. Message per `meerkatflow:pr` rules, naming the task.
```bash
W=<worktree absolute path>; cd "$W" || exit 1; D=<docs>/plans/<plan-name>; S=task-N
B=$(cat $D/$S.base); I=$D/$S.c.idx; cp "$(git rev-parse --git-path index)" $I
GIT_INDEX_FILE=$I git add -A 2>/dev/null; N=$(GIT_INDEX_FILE=$I git write-tree); rm -f $I
git diff --name-only -z $B $N | xargs -0 git add -- && git commit -m "<message>"
```
Log the commit hash next to the complete line. Fix rounds after the task loop (final review, verification) are committed the same way with their own `S`.

## Receiving review feedback (reviewer, user, external)

- Verify against the codebase before implementing.
- If any item is unclear, stop everything and ask about it first.
- Push back with evidence on wrong findings or unused-feature requests (YAGNI — grep for usages).
- If it conflicts with an earlier user decision, ask the user first.

## Finish

1. After all tasks, run full verification once, read output, compare with `$D/baseline.txt`. Only failures not in the baseline belong to this work. New failures: snapshot `S=final-R` → fix via that task's implementer → package and check the fix diff (the final review in step 4 covers it).
2. **Bundle the final diff.** The verifier defaults to bare `git diff`, which misses new files and mixes in pre-existing uncommitted changes. Bundle against the first task's base:
   ```bash
   W=<worktree absolute path>; cd "$W" || exit 1; D=<docs>/plans/<plan-name>
   B=$(cat $D/task-1.base); I=$D/final.now.idx; cp "$(git rev-parse --git-path index)" $I
   GIT_INDEX_FILE=$I git add -A 2>/dev/null; N=$(GIT_INDEX_FILE=$I git write-tree); rm -f $I
   { git diff --stat $B $N; git diff -U10 $B $N; } > $D/final-diff.md
   ```
3. **Constraint scan.** For every Global Constraint that can be checked mechanically, scan `final-diff.md`. Examples: a no-new-comments rule → `grep -nE '^\+\s*(//|/\*|\*|#|<!--)' $D/final-diff.md`; a no-test-files rule → `grep -nE '^\+\+\+ .*(__tests__|\.test\.|\.spec\.|_test\.)' $D/final-diff.md`. Violations are sent to one implementer for removal (snapshot/package as usual), then rebundle.
4. **Final code review.** Dispatch one reviewer on `final-diff.md` (opus; sonnet is enough for a 1–2 file diff). Template: [final-reviewer.md](final-reviewer.md)
   - Critical/Important: snapshot `S=final-review` → send all findings to **one** implementer → package with the same `S` → one scoped re-review.
   - Rule on anything left and log Rulings. Minor → `final: minor (deferred): ...`.
   - If anything was fixed, rebundle `final-diff.md` (step 2).
5. **Check verification state.** If `$W/.claude/verify/state.json` is `blocked` on a different criteria file, stop and report to the user (that verification comes first).
6. **Code-level verification.** Invoke `meerkatflow:verifying-against-criteria` with args:
   - Criteria file: plan absolute path
   - Change scope: "Use `$D/final-diff.md` as the change scope instead of `git diff`"
   - Constraints: plan's Global Constraints verbatim + "Do not deduct for the absence of anything the Global Constraints forbid"

   The verifier never edits code. The latest feedback is the largest-suffix `<plan-name>-feedback*.md` in `.claude/verify/`. If below 90:
   - Findings conflicting with Global Constraints are not sent; log a Ruling.
   - Criteria the verifier marked pending (not measurable here, or waiting on a user decision) go to the user, not to an implementer.
   - Snapshot `S=verify-R` → send all remaining findings to **one** implementer at once (sonnet; opus if design judgment is involved).
   - Package with the same `S`, read the fix, confirm no changes unrelated to findings. Rebundle `final-diff.md` and reverify against the same plan.
   - If the same finding stays unresolved twice in a row, stop and report to the user.
7. **Runtime verification.** Once code-level verification is ≥ 90, invoke `meerkatflow:verify-runtime` with the plan absolute path. It drives the running app itself and never edits code.
   - Fail: snapshot `S=runtime-R` → send the failed items (condition, expected, observed, evidence path) to **one** implementer → package with the same `S`, rebundle `final-diff.md`, rerun step 6 on the same plan when the fix changed behavior code (a markup- or style-only fix skips it), then rerun the failed items plus items sharing their code path.
   - Not measured items go to the user with their reason, not to an implementer.
   - At most 2 runtime fix rounds. Anything still failing after that, or the same item failing twice in a row, stop and report to the user.
8. **Record and report.** If the harness has persistent project memory, record only what the code and git history will not show: pitfalls found, user decisions (D#, only what the user chose) that outlive this work, questions still open, what was not measured. Progress and status stay in the ledger. Then the final message, in the user's language:
   - **Before → after:** old flow → what went wrong → new flow, in 2–4 plain sentences told through one concrete scenario (what the user does → what happens), readable without knowing the identifiers.
   - **Decisions I made:** every `Ruling:` line in the ledger, in order, with cost.
   - **Deferred Minors:** every `minor (deferred)` line.
   - **Runtime verification:** verdict, failed or not measured items with reasons, the result file path.
   - at-end: change summary and a commit message draft per `meerkatflow:pr`. per-task: the list of commits. Offer the PR body draft in one line.
9. Do not delete the workspace. The user cleans it up after committing.

## Inline mode

Use when the user asks for inline execution, the plan has 1–2 tasks, or there is no subagent tool. Same order, you implement. Per task: snapshot → implement → check verification output → ledger. Always do Finish steps 1–7. Final review: dispatch the reviewer as usual. You wrote the code, so your own read is not the review. Only with no subagent tool, read `final-diff.md` yourself against [final-reviewer.md](final-reviewer.md) and say in the report that it was a self-review.
