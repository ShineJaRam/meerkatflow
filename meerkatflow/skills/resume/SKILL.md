---
name: resume
description: Use when work moves between sessions — the user asks to write findings up so another session can continue, gives a handoff/investigation doc to continue from, or hands over work another agent session started (Codex, another Claude Code session; "continue this", "the other session froze", continue in worktree X). Writes the handoff document, or reconstructs approved scope and unfinished steps before touching code.
---

# Hand off and resume

## A. Writing a handoff (this session stops, another continues)

Save to `<docs>/handoffs/YYYY-MM-DD-<topic>.md`. No code changes. Sections, in the user's language:

- Conclusion: one or two sentences
- Evidence: file:line and measurements, each tagged `measured` / `code-only` / `inferred`
- Scope: in / out, as the user stated it
- Agent defaults: choices you made that the user did not. The next session must ask about them
- Open questions
- Acceptance criteria: observable result + how to check
- Next step: which skill continues

Tell the user the path.

## B. Continuing from a handoff document

1. Check the document's file:line claims still hold on the current code.
2. Ask its agent defaults and open questions as one numbered list.
3. 1–2 files → `meerkatflow:plan` with the document as source. Larger → `meerkatflow:spec`.

## C. Continuing another agent's session (no handoff document)

`git diff` alone does not show what was approved or what investigation was left half-done. Reconstruct first, act second.

### 1. Locate the session

- Codex (checked 2026-10): `~/.codex/sessions/YYYY/MM/DD/rollout-<time>-<uuid>.jsonl`. Match the worktree by `payload.cwd` on the `type === "session_meta"` line.
- Skip files whose first user_message starts with "The following is the Codex agent history whose request action you are assessing" (approval-review helper sessions).
- Claude Code: `~/.claude/projects/<project-slug>/<session-id>.jsonl`.
- Plan workspace: `<docs>/plans/<plan-name>/progress.md` ledger, if meerkatflow was used.

### 2. Extract the conversation

Codex:
```bash
node -e "require('fs').readFileSync(process.argv[1],'utf8').split('\n').filter(Boolean).map(JSON.parse).filter(l=>l.type==='event_msg'&&/^(user|agent)_message$/.test(l.payload.type)).forEach(l=>console.log('\n['+l.payload.type+']\n'+l.payload.message))" <file.jsonl>
```

Tool calls/results are `response_item` entries (`function_call` / `function_call_output`). Reasoning is encrypted; ignore it.

### 3. Establish state (show the user, then continue)

- **Approved scope:** the agent proposal right before the user's approval (a short "ok", "go ahead", "yes"). Only that is approved.
- **Done:** matched against the current `git diff --stat` and files.
- **Unfinished:** steps proposed/started but not done; investigation without a conclusion.
- **Cut-off:** did the last user message end mid-sentence?
- **Open decisions:** anything the user never answered.

Then continue with the matching meerkatflow skill (usually `meerkatflow:execute` from the ledger, or `meerkatflow:debug`). Do not redo finished work; do not expand beyond approved scope.
