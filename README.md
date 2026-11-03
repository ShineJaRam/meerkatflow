# meerkatflow

[한국어](README.ko.md)

A Claude Code plugin for a gated development flow: understand the request, design, plan, implement with subagents, then verify — first the code, then the running program. It also keeps code changes small and replies short.

Skills are called `meerkatflow:<skill>` (for example `meerkatflow:brainstorm`).

## Install

```
claude plugin marketplace add ShineJaRam/meerkatflow
claude plugin install meerkatflow@dev_meerkat
```

Requirements:

- `node` on PATH (the hooks print the rules file with node).
- `git`. Shell examples in the skills use bash (Git Bash on Windows).
- A subagent tool for `execute`. Without one it runs inline: the agent implements and reviews its own diff, and says so in the report.

The rules are active in every session from the start. Say "normal mode" to turn them off for the rest of the session.

## How a change flows

```
brainstorm → spec → plan → execute → verifying-against-criteria (90+) → verify-runtime
```

You do not have to start at the top. The agent picks the entry point:

| Request | Entry |
|---|---|
| A one- or two-file edit you spelled out | Edits directly, no skill |
| A new feature or behavior change with something still open | `brainstorm`, even if you named no skill. Say to skip it and it goes straight on |
| Intent is settled | `spec`. It first classifies the work: **spike** (an answer, not code), **bounded** (short design in chat, no document), **plan-only** (skip to `plan`) or **architectural** (spec document) |
| A small change with an investigation or handoff doc, or "plan only" | `plan` |
| A bug, error or failing build | `debug` |

Each arrow is a separate approval: agreeing with an idea does not approve the spec, and approving the spec does not approve the plan.

## Skills

| Skill | Use it when |
|---|---|
| `meerkatflow:brainstorm` | A new feature or behavior change still has open questions about what the user wants |
| `meerkatflow:spec` | The intent is agreed and the design needs to be settled and approved |
| `meerkatflow:plan` | An approved spec (or a short investigation doc) needs a task-by-task plan |
| `meerkatflow:execute` | Running an approved plan with one implementer subagent per task |
| `meerkatflow:verifying-against-criteria` | Scoring the finished code against the plan's acceptance criteria (90+ to pass) |
| `meerkatflow:verify-runtime` | Checking each runtime item in the running program, with evidence |
| `meerkatflow:review` | Reviewing the current diff without a plan |
| `meerkatflow:debug` | A bug, error or unexpected behavior, before any fix |
| `meerkatflow:measure` | Checking something in the running system (browser, API, database, jobs) |
| `meerkatflow:issue` | Turning a survey or investigation into issue drafts |
| `meerkatflow:pr` | Drafting a commit message or PR body |
| `meerkatflow:resume` | Handing work to another session, or picking up a handoff doc or another agent's session (Claude Code, Codex) |

## What it does

- **Asks before deciding for you.** Choices a spec leaves open ("A or B", "needs mockup") go to the user, never to an agent default. Choices the agent did make are labelled as its own, never as yours.
- **Commits only the way you choose.** At the start of `execute` you pick per-task commits (the agent commits each finished task on the feature branch, never pushes) or one commit at the end (the agent commits nothing; you commit). Outside `execute` it never commits unless you ask in that conversation.
- **Reviews once, at the end.** Each task gets a fresh implementer subagent that sees only its task's brief. The whole change gets one final review; a task gets its own review only when it is hard to undo (migration, auth, public contract, money).
- **Writes tests from your scenarios, before the code.** See [Test-driven flow](#test-driven-flow).
- **Does not claim "done" without evidence.** The code is scored against the plan's acceptance criteria and rechecked until it reaches 90. Then each runtime item is checked in the running program; screens are judged from screenshots the agent opens itself.
- **Says how it knows.** Survey and review findings are tagged `measured`, `code-only` or `inferred`, and each says who is hurt if it stays. Before reviewing or surveying it fetches and tells you if the branch is behind.
- **Keeps replies short.** The answer comes first. Yes/no questions are answered on the first line; details come when asked.
- **Keeps code small.** Before writing code: is it needed → does the repo have it → does the platform have it → does an installed dependency have it → does one line do it → only then the least code that works. Input checks, data-loss handling, security and accessibility are never cut for size.

## Test-driven flow

On by default since 1.1.0 for new features, behavior changes and bug fixes. Tests start from what the user does, not from the implementation.

| Step | What happens |
|---|---|
| `brainstorm` | Writes numbered scenarios: `S1: who → does what → what happens`. Every scenario is checked for its failure, empty-value and cancel branches; a branch it cannot fill becomes the next question |
| `spec` | Each scenario with behavior gets at least one acceptance criterion that names its `S#` |
| `plan` | A test map gives each scenario a level: `scenario` (the default: the user's action through the public interface — render and click, send a request, run the CLI; not necessarily end-to-end), `code` (only with a reason, such as input combinations), or `runtime` (visual behavior, hard-to-test code, no test runner; checked by `verify-runtime` instead) |
| `execute` | The implementer writes the tests first and must show each one failing for the intended reason before writing code. A test that passes at once means the behavior already exists; nothing is built for it |
| `debug` | Prefers a failing test as the repro, seen failing before the fix |
| `verifying-against-criteria` | Deducts for a scenario with no test, a test with no failing run, or a test no scenario connects to |

Not written: tests no scenario connects to, tests that mirror the implementation, coverage targets, or restructuring code just to make it testable. If the project has no test runner, the agent asks whether to add one and never installs one on its own.

Turning it off:

- `meerkatflow TDD: off` in `~/.claude/CLAUDE.md` turns it off everywhere, in a project's `CLAUDE.md` for that project; the project line wins.
- A rule of yours that forbids adding test files also turns it off. If you wrote `TDD: on` as well, `plan` asks once which to follow.
- Refactors, type-only, docs and config changes never use it, and neither does a one- or two-file edit you spelled out.

## Files it writes

All of these stay out of git. If a directory is not ignored, the agent adds it to `.git/info/exclude`.

| Path | What |
|---|---|
| `<docs>/specs/`, `<docs>/plans/` | Spec and plan documents, in the feature worktree |
| `<docs>/plans/<plan>/` | `execute` workspace: `progress.md` ledger, task briefs and reports, diffs, `baseline.txt` |
| `<docs>/surveys/`, `<docs>/handoffs/`, `<docs>/measure/` | Survey documents, handoff documents, measurement scripts and large results |
| `.claude/verify/` | Verification state, scored feedback and runtime results |

A resumed `execute` trusts the ledger: finished tasks are not run again. While a verification is below 90, `.claude/verify/state.json` is `blocked` and other criteria files are refused until that one passes or you drop it.

## Defaults and overrides

meerkatflow assumes nothing about your project. Everything below is a default; your project's `CLAUDE.md` or another plugin's session rules win.

| Setting | Default |
|---|---|
| Document language | The user's language |
| Agent documents `<docs>` | `.meerkatflow/` |
| Browser tool | Whatever browser automation the session has. A personal or project plugin with its own `measure` notes decides which tool and how |
| Worktree and branch | Sibling directory `feature-<key>`, branch `feature/<key>`. `<key>` is your issue key (asked for, never invented) or a short topic slug |
| Commit mode in `execute` | Asked once per run; a rule in `CLAUDE.md` can fix it. No answer → at the end, by you |
| Commit and PR format | The repository's recent history and PR template |
| Comment and test-file policy | None. Rules you have are passed to implementers through the plan's Global Constraints |
| Test-driven flow | On. See [Test-driven flow](#test-driven-flow) to turn it off |

Subagents never see session rules or `CLAUDE.md`. A rule implementers must follow reaches them only through the plan's Global Constraints, which `plan` fills in for you.

To override, put the rule in your `CLAUDE.md`, or ship a small plugin of your own with a session-start hook (and, for browser setups, a `measure` skill with your tool notes).

## Compared with superpowers + caveman + ponytail

meerkatflow covers the flow of superpowers, the short replies of caveman and the small-code rules of ponytail in one plugin. superpowers, caveman and ponytail were measured from each project's repository on 2026-10-30 (default branch); meerkatflow at 1.1.0:

| | superpowers + caveman + ponytail | meerkatflow |
|---|---|---|
| Skills listed in every session | 43 | 12 |
| Skills with overlapping jobs (review ×4, debugging/investigation ×2, "verify before done" ×2, help ×2) | 10 | 0 |
| Fixed context per session: skill descriptions + session-start rules | ≈ 18,700 chars (≈ 4,700 tokens) | ≈ 11,700 chars (≈ 2,900 tokens), **−37%** |
| Rules injected into every subagent | ≈ 7,500 chars (≈ 1,900 tokens) per subagent | 0 (implementers get the rules they need inside their task prompt) |
| Commits | the superpowers implementer commits each task | your choice per run: per task, or none (you commit at the end) |
| Runtime check of the finished change, with evidence per item | not included | `verify-runtime` |

How the numbers were taken: description lengths from every `skills/*/SKILL.md`; session-start rules from what each hook injects (superpowers: `using-superpowers`; caveman and ponytail: the body of their main skill, which their hooks load — an approximation); tokens ≈ characters ÷ 4. Not measured: per-prompt reminders, and outcome quality (diff size, tokens spent, rework on real tasks). A side-by-side run on the same tasks has not been done yet.

## Contributing

Edit this repository only, never `~/.claude/plugins/cache`.

1. Edit the files.
2. Bump `version` in `meerkatflow/.claude-plugin/plugin.json`.
3. Add a line to `CHANGELOG.md`.
4. `claude plugin marketplace update dev_meerkat`, then `claude plugin update meerkatflow@dev_meerkat`.

What goes in:

- Only rules that hold in any project. Personal taste and project specifics belong in your own plugin or `CLAUDE.md`.
- Skills do not refer to other plugins' skills by name.
- No company code, internal paths, API or component names, or real commit titles as examples.
- Facts that change quickly (tool behavior, tool names) carry the month they were checked.
- A skill nobody has used for a quarter is removed.

## Credits

meerkatflow was written from scratch, inspired by ideas from [superpowers](https://github.com/obra/superpowers), [caveman](https://github.com/JuliusBrussee/caveman) and [ponytail](https://github.com/DietrichGebert/ponytail). See [NOTICE.md](NOTICE.md).
