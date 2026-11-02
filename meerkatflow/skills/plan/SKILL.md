---
name: plan
description: Use when an approved spec, a handoff doc for a small change, or clear requirements exist for a multi-step change, before touching code. Writes a task-by-task implementation plan with interfaces, verification steps, a runtime checklist and acceptance criteria.
---

# Plan

Turn an approved spec (or, for a 1–2 file change, a written investigation/handoff doc) into a task-by-task implementation plan. Assume the reader is a capable developer (subagent) who does not know this codebase.

With no spec, put `**Source:** <doc path>` where `**Spec:**` goes and write the acceptance criteria in the plan itself.

Save to the same feature worktree as the spec: `<docs>/plans/YYYY-MM-DD-<feature>.md`.

## 0. Open choices first

Before writing, collect every choice the spec left open: its "Open questions" section and any "A or B", "TBD", "to be confirmed", "needs mockup" wording anywhere in it. Ask the user as one numbered text list and record answers as user decisions in the plan. Pure implementation details you decide are marked "agent default", never "user decision".

Conditional tasks ("if X exists, else skip"): the gate needs evidence X actually exists, not that code could support X. No evidence → the task stays out.

## 0.1 TDD value

Read `meerkatflow TDD: on|off` from `CLAUDE.md` and from session rules the user or another plugin added; a project `CLAUDE.md` beats the global one.

- No line, and no user rule forbids adding test files → on.
- No line, and a user rule forbids adding test files → off, without asking. Say why in one line of your hand-off.
- The user wrote `on`, and a user rule forbids adding test files → ask once which one to follow. Record the answer as a user decision. If on, leave that rule out of Global Constraints; if off, write `TDD: off` and keep the rule.
- `off` → off.
- Work with no behavior change (refactor, types, docs, config) → off, whatever the line says. Say why in one line of your hand-off.

Write `TDD: on` or `TDD: off` into Global Constraints. When off, write no Test map and skip the rest of this section.

When on:
- Scenarios (`S1: who → does what → what happens`) come from the spec or brainstorm. If the source has none, including when brainstorm was skipped, list them and get the user's confirmation before writing the plan.
- Find the project's test runner. If there is none, ask whether to add one, and never install one on your own. If not added, every row is `runtime`; say so in the plan and in the final report.

## 1. File structure first

Before splitting tasks, list files to create/modify and each file's responsibility.
- Follow existing patterns. Do not restructure large files on your own.
- Files that change together belong to the same task.
- Contract first: a change to an interface another side consumes (API, schema, event) is its own early task. State what happens to old clients and existing rows, and how to roll back.

## 2. Task size

- Task = smallest unit that can be checked on its own (its files, its verification command).
- Tasks are reviewed once, at the end. Add `**Review:** required` to a task that is hard to undo or widely shared (data migration, auth/permissions, public contract, money).
- Fold setup/config work into the task that needs it.
- Bundle many same-shaped small edits (same replacement across files) into one task.

## 3. Document format

Headings and body in the document's language. Keep `Task N`, the `AC#` ids, and the literal headings `Global Constraints` and `Acceptance Criteria` (other skills look for them).

````markdown
# <feature> implementation plan

**Spec:** <spec path>
**Goal:** <one sentence>
**Approach:** <2–3 sentences>

## Global Constraints
- <global rules from the spec, values copied verbatim> (e.g. token names, copy text, prohibitions)
- <standing rules from session rules / CLAUDE.md that implementers must obey, copied verbatim> (e.g. comment policy, test-file policy, which tool runs verification commands)
- Implementers do not commit

## Runtime checklist
<up to 5 inputs/situations typecheck cannot catch and the user will hit first. Each line: condition → expected behavior → owning task. Nothing observable changes at runtime (pure refactor, types only) → `none — <reason>`>

## Test map
<TDD on only>
| Scenario | Level | Reason | Test | Task |
|---|---|---|---|---|
| S1 | scenario | - | path/file.test.ts "name" | Task 1 |
| S1 | code | input combinations | path/parse.test.ts "name" | Task 1 |
| S2 | runtime | visual behavior | Runtime checklist line 2 | Task 2 |

---

### Task N: <name>

**Review:** required  <only on tasks that need a task review; otherwise omit the line>

**Files:**
- Modify: `exact/path/file.ext:120-145`
- Create: `exact/path/new.ext`

**Interfaces:**
- Consumes: <from earlier tasks — exact names/types>
- Produces: <used by later tasks — exact names/types>

**Change:**
<what and how. Code steps carry real content in code blocks>

**Verification:**
- Run: `<typecheck/lint/test command>` → Expected: <result>
- Runtime: <screen path + action, or request / job → expected response and data state> (if applicable)

## Acceptance Criteria
- AC1: <spec AC copied, with owning Task number>
````

Subagents never see the session rules. A standing rule that is not written into Global Constraints does not exist for the implementer.

Test map levels:
- `scenario` is the default: reproduce the user's action through the public interface (render and interact with a component, send a request, call a CLI). It does not mean end-to-end.
- `code` needs one reason: `input combinations` (date boundaries, parsers, mappers a scenario cannot cover), `no outward behavior` (a shared util or internal function changes and no scenario can express it), or `narrowing a failure` (a scenario test fails and the location is unclear; this reason appears only during implementation, which adds the row then).
- `runtime` needs a reason (visual behavior, existing code that is hard to test, no runner) and a Runtime checklist line. Do not add tasks that restructure code to make it testable.
- A test no scenario connects to is not written. Tests that mirror the implementation are not scenario tests. Coverage numbers are not a goal.
- Test files go in the owning task's Files.

On a first plan with TDD on, tell the user in one line of your hand-off that `meerkatflow TDD: off` in `CLAUDE.md` turns it off.

## 4. No placeholders

These are plan failures:
- "TBD", "later", "handle appropriately", "handle edge cases"
- "Same as Task 3" (implementers never see other tasks — repeat it)
- References to functions/types no task defines
- Code steps stating a goal without the method

## 5. Self-check

1. Every spec requirement/AC has an owning task. Otherwise add a task.
2. No placeholders.
3. Earlier Produces match later Consumes in name and type.
4. Every runtime checklist line has an owning task.
5. With TDD on: every scenario has a Test map row, every `code` and `runtime` row has a reason, and every `runtime` row has a Runtime checklist line.

Fix problems immediately.

Then dispatch the document reviewer, unless the plan has only 1–2 tasks (the self-check is enough). Template: [doc-reviewer.md](doc-reviewer.md)

## 6. Hand off

After handling the review, give the path and ask for review. Default execution is subagents (`meerkatflow:execute`). If there are only 1–2 tasks or they are tightly coupled, recommend inline execution with a one-line reason. The user asking for inline wins.
