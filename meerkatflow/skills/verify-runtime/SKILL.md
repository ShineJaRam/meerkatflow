---
name: verify-runtime
description: Use after code-level verification passes (meerkatflow:verifying-against-criteria ≥ 90), or when asked to confirm in the running program that a change behaves as a plan or spec says. Drives the real program — screens through a browser and judged from screenshots you open yourself, APIs, jobs and data through real requests and read-back queries — and records pass / fail / not measured with evidence for each runtime checklist line and runtime acceptance criterion. Verifier stance only — does not implement.
---

# Verify runtime

Code-level verification says the code should work. This says it does: you run the program and watch each promised behavior happen. Tool mechanics (finding the instance, driving the browser, reading state, service checks) live in `meerkatflow:measure`; load it, and any personal or project measure notes, before starting.

**Verifier, not implementer.** Never edit code. Failures go back to whoever called you.

## 1. Collect the items

From the criteria file (plan or spec):
- every `Runtime checklist` line
- every acceptance criterion whose "how to check" is a screen, an action, a request or a data state
- criteria the code-level verifier marked pending as not measurable from code

One item = one condition → one expected observation. Split items that bundle several.

No items (the plan's Runtime checklist says `none` and no criterion needs the running program) → do not start anything. Write the result file with verdict `no runtime items` and the plan's reason, and stop.

## 2. Plan the run

- Find the instance running this worktree's code (`meerkatflow:measure`, Find the right server). State the URL or base URL.
- Group items by the state they need (page, data, user role) so each state is set up once.
- Items needing data that does not exist: create it only if it is yours to create and remove (`meerkatflow:measure`, Service checks: Writes). Otherwise the item is not measured, with what data it needs.
- Several pages or states, or an item you will rerun after a fix: write the run as one script file `<docs>/measure/<criteria-basename>-runtime.<ext>` and rerun it unchanged.

## 3. Observe each item

For each item, in the running program:
1. Reach the state the condition describes, the way a user or caller would (clicks and input, a real request, a real job).
2. Capture the observation: the value read from the screen or response, console errors, failed requests, and for anything visual a screenshot saved to a file.
3. **Judge visuals yourself.** Open each screenshot you saved (read the image file) and compare it with the expected result: the element is there, the text and values are visible and not cut off, layout and alignment match, nothing that was visible before disappeared.
4. Record the result: `pass` (observed matches expected), `fail` (observed differs: say what you saw), or `not measured` (reason and what is needed). "Did not try" is not a reason.

Only what no tool in this session can observe goes to the user: frame timing in a throttled tab, screens behind permissions or accounts you do not have, hardware. Say exactly what they should look at and what the expected result is.

## 4. Coverage

List the variants the items imply (empty / one / many, item types, roles) and which you ran, per `meerkatflow:measure` Report.

## 5. Result

Write `.claude/verify/<criteria-basename>-runtime.md` (same directory and git exclusion as `meerkatflow:verifying-against-criteria`; on a rerun add `-1`, `-2`, … and never overwrite):

```markdown
# <criteria basename> runtime verification

## Verdict
<pass | fail | partly measured> — <one sentence>
Instance: <URL / base URL>, worktree <path>

## Items
| # | Condition | Expected | Observed | Evidence | Result |
|---|---|---|---|---|---|
| 1 | <state + action> | <expected> | <what you saw> | <screenshot path / value / log> | pass / fail / not measured |

## Not measured
- <item>: <reason> — needs <what>

## Coverage
<variants run / not run>

## Notes
<script path, data created and removed, anything that looked off outside the items>
```

Verdict:
- **pass**: every item passed. Items not measured are allowed only with a reason; list each for the user.
- **fail**: at least one item failed. The caller fixes and reruns the failed items plus any item sharing their code path.

Headings and body in the user's language; keep this structure.
