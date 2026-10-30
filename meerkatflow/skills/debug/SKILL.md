---
name: debug
description: Use for any bug, error, failing type check or build, or behavior that differs from what was expected, before any fix is proposed. Root cause first, one hypothesis at a time, stop after 3 failed fixes.
---

# Debug

**Iron rule: no fix before root-cause investigation.** Fixing only the symptom is failure.

## Phase 1: Investigate

1. **Read the error to the end.** Stack, file:line, code. Warnings too.
2. **Reproduce.** Exact steps. If it will not reproduce, gather more data. No guessing. A bug seen in the running app is reproduced there (`meerkatflow:measure`) before you report a cause. A browser repro that is hard to trigger or will be repeated goes in a script file (`meerkatflow:measure` section 2) and is rerun unchanged after the fix. A cause found by reading code alone is a hypothesis: label it so and say what would confirm it.
3. **Check recent changes.** `git diff`, `git log`, dependency/config/environment differences.
4. **Instrument each boundary.** With layers like client ↔ API ↔ service ↔ storage, log what enters and leaves each boundary once to find the breaking layer first (request body/response, logs at each side).
5. **Trace the value backwards.** Where did the bad value come from → who called with it → up to the source. Fix at the source.

## Phase 2: Compare

- Find similar **working** code in the same codebase.
- Read reference implementations fully. No skimming.
- List every difference between working and broken. No "that can't matter".

## Phase 3: One hypothesis

- Write "X is the cause, because Y".
- Test with the smallest single change. One variable at a time.
- Wrong → new hypothesis. Never stack fixes on fixes.
- If you do not know, say so.

## Phase 4: Fix

1. Secure a repro procedure (failing test, typecheck command, browser steps, console snippet, request script).
2. One fix for the cause. No "while I'm here" improvements. When several candidate causes show up, fix only the one shown to produce the reported symptom; the rest stay listed as unconfirmed until evidence shows they matter.
3. Confirm with the same repro.
4. Report before → after in the session rules' change-explanation shape (old flow → what went wrong → new flow), then the evidence.

## When stuck

Open [techniques.md](techniques.md):
- Call chain too deep to find the source → 1. Backward value tracing
- Breaks only sometimes, timing suspected → 2. Intermittent repro
- Source fixed but the same kind of value could leak again → 3. Defense in depth
- "It used to work" → 4. When did it break
- Tests fail only when run together → 5. Polluting test

## Rule of 3

Stop after 3 failed fixes. If each fix surfaces a new problem elsewhere, it is structural. Discuss the structure with the user before a 4th attempt.

## When the cause is outside your editable scope

If you conclude the cause is in code you may not edit (another team's service, server version, data):
- Show evidence (request/response, server logs, versions).
- Report separately what can be done inside your scope (defensive handling, error copy) and what goes to the owner.
- Never edit the out-of-scope code.

## Stop signals

Return to Phase 1 when you think:
- "Let me just change X and see"
- "Probably X, let's fix it"
- "Change several things and rerun"
- The user says "is that not happening?", "stop guessing", "are you stuck?"
