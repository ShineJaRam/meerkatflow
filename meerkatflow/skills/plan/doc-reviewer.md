# Document reviewer dispatch template (shared by spec and plan)

```
Agent (general-purpose)
  description: "Review <spec|plan>: <title>"
  model: sonnet
  prompt: |
    Review a <spec|plan> document. Find where the person receiving it would get stuck or build the wrong thing.

    ## Input
    Document: <DOC_FILE>
    (plan only) Source spec: <SPEC_FILE>
    Code location: <worktree absolute path>

    Read-only. Do not edit the document or code. Do not dispatch subagents.
    Confirm that files/functions/components the document references actually exist (Grep/Read). Do not search beyond that.

    ## For a spec, check
    - Requirements readable two ways, contradictions between sections
    - Ungradable acceptance criteria ("works well", no verification method)
    - Items in the user-decisions section that look like defaults the user never chose
    - Items straddling scope (in/out) without saying which side
    - Independent subsystems mixed beyond what one plan can handle

    ## For a plan, check
    - Spec requirements/ACs with no owning task
    - Placeholders: TBD, "appropriately", "same as Task N", goal-only steps without method
    - Paths/line ranges/functions that do not exist (verify in code)
    - Produces ↔ Consumes name/type mismatches, references no task defines
    - Places a task cannot be implemented from its own text (needs another task's body)
    - Missing or unrunnable verification commands (e.g. uses a tool the project forbids or does not have)
    - Steps conflicting with Global Constraints

    Do not flag taste or style. Only risks that make an implementer build it wrong.
    Do not propose widening scope. Something outside the document's scope that looks wrong is [Minor], never [Blocking].

    ## Format (English)
    Start with the list.
    - [Blocking] <location (section / Task N)>: <problem> → <fix>
    - [Minor] <location>: <problem>
    If none, one line: "No issues".
```

## Handling

- Fix Blocking items immediately. Then dispatch the same reviewer once more (check only the fixed parts). Max 2 rounds.
- Findings that need a user decision (open choices) are not settled by you; move them to the open-questions section.
- Blocking items left after 2 rounds, plus Minor items, are listed in the user's language when you ask the user to review.
