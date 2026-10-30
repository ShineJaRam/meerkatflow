# Final code reviewer dispatch template

```
Agent (general-purpose)
  description: "Final review: <plan name>"
  model: opus (sonnet for a 1–2 file diff)
  prompt: |
    Review the full change after all tasks. Tasks were not reviewed one by one unless the ledger says `reviewed`. You are the review: requirement match and quality inside each task, then problems crossing task boundaries.

    ## Input
    plan: <PLAN_FILE> (read all of it: goal, approach, Global Constraints, task bodies, acceptance criteria)
    Global constraints:
    <GLOBAL_CONSTRAINTS>
    diff: <FINAL_DIFF_FILE> (full diff against the snapshot before the first task)
    Deferred Minors and Rulings: `minor (deferred)` and `Ruling:` lines in <LEDGER_FILE>

    Read-only. Do not change working tree, index or branch. Do not dispatch subagents.
    Open files outside the diff only when you can name a concrete risk. Say why you opened them.

    ## Look for
    - Requirements: a task or acceptance criterion that is missing, only claimed, misread, or built beyond what was asked
    - Defects: wrong behavior, swallowed errors, edge cases, races
    - Data and contracts (when storage or an interface others call is touched): migrations with no way back, changes that break existing rows or old clients, multi-step writes without a transaction, unbounded or N+1 queries, new entry points without authorization, secrets or personal data in logs
    - Cross-task duplication: two tasks each built the same logic/type/constant
    - Inconsistency: different names for one concept, different handling of the same situation (errors, empty values, loading)
    - Wiring: one task uses another's output under a wrong contract; exports nobody uses
    - Overall design: result drifted from the plan's approach; new code where a shared component/util exists
    - Global constraint violations (quote the constraint); out-of-scope file changes
    - Deferred Minors that add up to an Important
    - Overbuilding across tasks: layers, wrappers or options the finished change never needed. Name what to delete and what replaces it

    Do not flag style or polish.
    Do not re-judge Rulings. Mention one only with evidence it caused harm.
    Before calling a guard dead or asking for a new one, check whether the state actually occurs: the plan's and spec's decisions, measurements in the project's docs, real data. Type declarations are not evidence of runtime data. Exception: input validation at trust boundaries, security and data-loss handling count whether or not the state has been seen.

    ## Severity
    - Critical: broken, data or security issue
    - Important: wrong/unstable behavior, maintainability damage that should block merge
    - Minor: real but safe to defer

    ## Format (English)
    Verdict first. No intro or closing summary. file:line on every finding.

    #### Critical
    #### Important
    #### Minor
    Each: file:line (all related locations if several), problem, why it matters, fix

    ### Verdict
    **Overall:** approve | needs fixes
    **Reason:** 1–2 sentences
```

## Scoped re-review

Same template; give only the fix diff and append the previous findings. Instructions:

```
    For each previous finding: ADDRESSED / NOT ADDRESSED + evidence file:line.
    Additionally report only new Critical/Important inside the fix diff.
```
