# meerkatflow

[한국어](README.ko.md)

A Claude Code plugin for a gated development flow: understand the request, design, plan, implement with subagents, then verify — first the code, then the running program. It also keeps code changes small and replies short.

Skills are called `meerkatflow:<skill>` (for example `meerkatflow:brainstorm`).

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
| `meerkatflow:resume` | Handing work to another session, or picking it up |

## What it does

- **Asks before deciding for you.** Approving an idea, a spec and a plan are separate steps. Choices a spec leaves open ("A or B") go to the user, never to an agent default.
- **Commits only the way you choose.** At the start of `execute` you pick per-task commits (the agent commits each finished task on the feature branch, never pushes) or one commit at the end (the agent commits nothing; you commit). Either way the whole change gets one review at the end (per-task review only for tasks that are hard to undo).
- **Does not claim "done" without evidence.** The code is scored against the plan's acceptance criteria and rechecked until it reaches 90. Then each runtime item is checked in the running program; screens are judged from screenshots the agent opens itself.
- **Keeps replies short.** The answer comes first. Yes/no questions are answered on the first line; details come when asked.
- **Keeps code small.** Before writing code: is it needed → does the repo have it → does the platform have it → does an installed dependency have it → does one line do it → only then the least code that works.

## Compared with superpowers + caveman + ponytail

meerkatflow covers the flow of superpowers, the short replies of caveman and the small-code rules of ponytail in one plugin. Measured from each project's repository on 2026-10-30 (default branch):

| | superpowers + caveman + ponytail | meerkatflow |
|---|---|---|
| Skills listed in every session | 43 | 12 |
| Skills with overlapping jobs (review ×4, debugging/investigation ×2, "verify before done" ×2, help ×2) | 10 | 0 |
| Fixed context per session: skill descriptions + session-start rules | ≈ 18,700 chars (≈ 4,700 tokens) | ≈ 11,400 chars (≈ 2,850 tokens), **−39%** |
| Rules injected into every subagent | ≈ 7,500 chars (≈ 1,900 tokens) per subagent | 0 (implementers get the rules they need inside their task prompt) |
| Commits | the superpowers implementer commits each task | your choice per run: per task, or none (you commit at the end) |
| Runtime check of the finished change, with evidence per item | not included | `verify-runtime` |

How the numbers were taken: description lengths from every `skills/*/SKILL.md`; session-start rules from what each hook injects (superpowers: `using-superpowers`; caveman and ponytail: the body of their main skill, which their hooks load — an approximation); tokens ≈ characters ÷ 4. Not measured: per-prompt reminders, and outcome quality (diff size, tokens spent, rework on real tasks). A side-by-side run on the same tasks has not been done yet.

## Defaults and overrides

meerkatflow assumes nothing about your project. Everything below is a default; your project's `CLAUDE.md` or another plugin's session rules win.

| Setting | Default |
|---|---|
| Document language | The user's language |
| Agent documents `<docs>` | `.meerkatflow/` (excluded from git) |
| Verification state and feedback | `.claude/verify/` (excluded from git) |
| Browser tool | Whatever browser automation the session has. A personal or project plugin with its own `measure` notes decides which tool and how |
| Worktree and branch | `feature-<key>` / `feature/<key>` |
| Commit mode in `execute` | Asked once per run; a rule in `CLAUDE.md` can fix it. No answer → at the end, by you |
| Commit and PR format | The repository's recent history and PR template |
| Comment and test-file policy | None. Rules you have are passed to implementers through the plan's Global Constraints |

To override, put the rule in your `CLAUDE.md`, or ship a small plugin of your own with a session-start hook (and, for browser setups, a `measure` skill with your tool notes).

## Requirements

- `node` on PATH (the hooks print the rules file with node).
- `git`. Shell examples in the skills use bash (Git Bash on Windows).

## Install

```
claude plugin marketplace add <path or git URL of this repository>
claude plugin install meerkatflow@dev_meerkat
```

## Changing it

Edit this repository only, never `~/.claude/plugins/cache`.

1. Edit the files.
2. Bump `version` in `meerkatflow/.claude-plugin/plugin.json`.
3. Add a line to `CHANGELOG.md`.
4. `claude plugin marketplace update dev_meerkat`, then `claude plugin update meerkatflow@dev_meerkat`.

## What goes in

- Only rules that hold in any project. Personal taste and project specifics belong in your own plugin or `CLAUDE.md`.
- Skills do not refer to other plugins' skills by name.
- No company code, internal paths, API or component names, or real commit titles as examples.
- Facts that change quickly (tool behavior, tool names) carry the month they were checked.
- A skill nobody has used for a quarter is removed.

## Credits

meerkatflow was written from scratch, inspired by ideas from [superpowers](https://github.com/obra/superpowers), [caveman](https://github.com/JuliusBrussee/caveman) and [ponytail](https://github.com/DietrichGebert/ponytail). See [NOTICE.md](NOTICE.md).
