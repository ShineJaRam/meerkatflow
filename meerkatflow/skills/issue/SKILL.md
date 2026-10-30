---
name: issue
description: Use when turning a survey or investigation (codebase audit, adoption audit, improvement candidates, QA findings) into an issue breakdown or issue-tracker ticket drafts, or when asked to suggest an issue title. Groups by component, sizes issues, and drafts tickets without creating them until approved.
---

# Issue breakdown

Investigation and documents only. No code changes.

## 1. Survey document

Save to `<docs>/surveys/YYYY-MM-DD-<topic>.md`. Each item: file:line, current state, target, harm (who hits what if it stays), evidence tagged `measured` / `code-only` / `inferred` (session rules).

## 2. Grouping rules

Defaults.

- **One issue per component or module.** Never split a large one into (1)/(2); use sub-headings for items with different confidence/priority.
- Never group by screen (e.g. one dashboard) or by work type (e.g. "cleanup"). Distribute items to their component issues.
- **Small components get bundled**: aim for ~5+ changed files per issue. Pair within the same family (inputs together, overlays together) or shared files/prerequisites. Title lists every bundled component; sub-heading per component inside. The 5-file threshold is a default — say so in the report.
- Items without a component: the user decides where they go. Ask.
- Do not create a "0th" prerequisite issue. Fold prerequisites into the first issue that needs them.

## 3. Draft format

In the language the tracker's existing tickets use.

```markdown
### <title>
- Scope: <components / file count>
- Background: <why>
- Work:
  #### <component A>
  - <item> — `path:line`
- To confirm: <inferred items, items needing a user decision>
```

Show the breakdown as a markdown table in chat first (title / components / file count / confidence).

## 4. Creating tickets

- Create tickets only after explicit approval of the breakdown. Confirm the project/repo and issue type first.
- Use whatever tracker tool the session has (Jira or Linear MCP tools, `gh issue create`). Deferred tools load via ToolSearch. Create one at a time; report each key or URL.
- No tracker tool → hand over the drafts as text.
- If the harness has persistent project memory, record the final breakdown and keys.
