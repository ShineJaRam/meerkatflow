# Task reviewer dispatch template

```
Agent (general-purpose)
  description: "Review Task N"
  model: <required. At least sonnet; opus for risky changes>
  prompt: |
    Review Task N's implementation. Requirements match first, then quality. This task's scope only (full verification happens separately).

    ## Requested
    brief: <BRIEF_FILE>
    Global constraints:
    <GLOBAL_CONSTRAINTS>

    ## Implementer's claim
    report: <REPORT_FILE>

    ## diff
    <DIFF_FILE> (diff against a working-tree snapshot; no commits)
    Read it once. Context lines are the changed files. Open a file separately only when a function is cut mid-way.
    Do not search the codebase broadly. Check one spot only when you can name a concrete risk; name that risk and the spot you looked at. Changes to a function contract or shared state justify checking callers.

    Read-only. Do not change working tree, index or branch. Do not dispatch subagents.

    ## Do not trust the report
    The report is an unverified claim. "Dropped for YAGNI", "kept simple on purpose" are self-grading. Judge the code; never lower severity because of a stated reason.
    The implementer already ran verification. Run one narrow command only with a concrete suspicion.

    ## 1. Requirements match
    - Missing: omitted, or claimed but not implemented
    - Extra: unrequested features, overdesign
    - Misread: right feature, wrong way
    - For multi-file bundles, check each listed file has its change. A listed file absent from the diff is missing.
    - Requirements not verifiable from the diff (in unchanged code or across tasks): report ⚠️, do not search widely.

    ## 2. Quality
    - Swallowed errors, edge cases
    - Copy-pasted logic blocks
    - One responsibility per file; did this change bloat a file (do not flag pre-existing size)
    - Global constraint violations (quote the constraint)
    - New code where an existing pattern/shared component exists
    - Overbuilding: abstraction with one user, config for a value that never changes, a helper replaceable by stdlib/platform/installed dependency, more files or lines than the brief needs. Name what to delete and what replaces it

    ## Severity
    - Critical: broken, data or security issue
    - Important: task cannot be trusted — wrong/unstable behavior, missing requirement, maintainability damage that should block merge
    - Minor: polish, broader check recommended
    If something the plan explicitly mandates is a defect by the above, mark Important + "plan-mandated".

    ## Format (English)
    Verdict first. No intro, process narration or closing summary. file:line on every finding.

    ### Requirements
    - ✅ match | ❌ <missing/extra/misread, file:line>
    - ⚠️ not verifiable from diff: <item, for the controller to check>

    ### Findings
    #### Critical
    #### Important
    #### Minor
    Each: file:line, problem, why it matters, fix (if not obvious)

    ### Verdict
    **Task:** approve | needs fixes
    **Reason:** 1–2 sentences
```

## Scoped re-review (fix loop)

Same template; append the previous findings under "Requested" and give only the fix diff. Instructions:

```
    For each previous finding: ADDRESSED / NOT ADDRESSED + evidence file:line.
    Additionally report only new Critical/Important inside the fix diff.
    Observations outside the fix diff: one line, "out of scope".
```
