---
name: spec
description: Use after meerkatflow:brainstorm (or when the request already settles intent) to turn the agreed intent and approach into the right output before any code. Classifies spike/bounded/plan-only/architectural; the architectural path produces a spec document with gradable acceptance criteria.
---

# Spec

Turn the intent and approach agreed in `meerkatflow:brainstorm` into a design the user approves. If intent is still unclear (no brainstorm summary, and the request leaves something non-trivial open), go back to `meerkatflow:brainstorm` first.

## 1. Classify the path (say it out loud before the first question)

| Path | When | Output |
|---|---|---|
| spike | "Is it possible?" investigation. The answer is the deliverable, not code | 2–3 sentence investigation plan → nod → investigate → recommend. Mark any code as throwaway |
| bounded | Narrow change to an existing flow where how to do it is still open (a flag, one-file edit) | Short design in chat → implement **after explicit approval**. No document |
| plan-only | A written investigation/handoff doc already fixes scope and checks and the change is 1–2 files, or the user asks for "plan only" | No spec. Go to `meerkatflow:plan` with that doc as the source |
| architectural | New screen/module, structural change across components, change to an interface others use | Spec document → approval → `meerkatflow:plan` |

When unsure, pick the heavier path. If hidden complexity appears mid-work, stop and escalate one level. Never downgrade on your own; the user may.

## 2. Remaining decisions

Intent, goals and the chosen approach come from the brainstorm summary; do not ask them again. What is left are implementation choices the user should still make.

Question rules:
- Decisions that do not depend on each other: one numbered list in a single message, lettered options each, your recommendation first, so the user can answer `1 A, 2 B`. Ask one at a time only when a later question depends on an earlier answer.
- Each option is one complete sentence a reader understands without the codebase: no abbreviations, ids you coined or bare file names.
- If the request mixes independent subsystems, propose splitting first.

## 3. Design (architectural)

- Build on the approach the user picked in brainstorm. If brainstorm was skipped, lay out 2–3 approaches with a recommendation in a message of their own first.
- Present the design section by section, confirm each. Keep it proportional to the size.
- Do not propose unrelated refactors. Only fix problems in the way of this work.

## 4. Spec document

Save to the feature worktree: `<docs>/specs/YYYY-MM-DD-<topic>.md`.

`<docs>` is gitignored, so it exists per worktree. A spec written on main is invisible from the implementation worktree. For architectural work, create the worktree first and write there. If already written on main, move it.

### Creating the worktree

1. Find the issue key (e.g. `PROJ-123`) in the request/conversation. If the project uses issue keys and none was given, ask. Never invent one. If the user says the key comes later, write the spec in the main checkout's `<docs>` and move it once the worktree exists. No issue tracker → use a short kebab-case topic slug as `<key>`.
2. If `git worktree list` already has `feature/<key>`, use that path and skip steps 3–4.
3. Create it from the main branch, as a sibling of the main checkout named `feature-<key>` (a project branch-naming rule wins):
   ```bash
   M=$(git worktree list --porcelain | sed -n '1s/^worktree //p'); K=<key>
   git -C "$M" worktree add "$(dirname "$M")/feature-$K" -b "feature/$K" <main branch>
   ```
   If `feature/<key>` exists only on the remote, drop `-b`: `git -C "$M" worktree add <path> "feature/$K"`.
4. Dependencies: if the project has a worktree setup rule (session rules, CLAUDE.md), follow it. Otherwise install them the way the project normally does.
5. From now on start every Bash command with `cd <worktree absolute path>` (cwd resets per call). Tell the user the worktree path in one line.

Required sections. Write headings and body in the document's language, but keep the ids `D#` / `AC#` and the literal words `Acceptance Criteria` in that heading (other skills look for them):

```markdown
# <title>

## Background and goal
## Scope (in / out / anti-goals)
## Approaches
- <chosen approach> — chosen because …; <rejected approach> — rejected because …
## Design
## User decisions
- D1: <decision> — <rationale>
## Open questions
## Acceptance Criteria
- AC1: <observable result> — how to check: <typecheck or test command / screen·action / request → response and data state / code location>
```

- Acceptance criteria must be gradable. No "works well". State the action and the observable result.
- "User decisions" holds only what the user chose. Defaults you filled go to "Open questions" or get explicit confirmation.

## 5. Self-check, doc review, then user review

Right after writing, scan once and fix immediately:
1. No TBD/TODO/vague requirements
2. No contradictions between sections
3. Fits in one plan
4. No sentence readable two ways

Then dispatch the document reviewer. Template: [../plan/doc-reviewer.md](../plan/doc-reviewer.md)

After handling the review, give the path and ask for review. Once approved, go to `meerkatflow:plan`. No other implementation skill.

## Gates

- Approval is valid only for the step presented. Idea approval ≠ spec approval ≠ plan approval.
- Before a gate, read-only investigation only.
