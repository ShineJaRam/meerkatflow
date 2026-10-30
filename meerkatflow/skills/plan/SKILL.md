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

Fix problems immediately.

Then dispatch the document reviewer, unless the plan has only 1–2 tasks (the self-check is enough). Template: [doc-reviewer.md](doc-reviewer.md)

## 6. Hand off

After handling the review, give the path and ask for review. Default execution is subagents (`meerkatflow:execute`). If there are only 1–2 tasks or they are tightly coupled, recommend inline execution with a one-line reason. The user asking for inline wins.
