---
name: brainstorm
description: Use before any new feature, component change or behavior change, even when the user did not ask for a skill — to find out what the user actually wants and why before anything is designed or built. Asks one open question at a time, plays understanding back, lays out approaches for the user to pick, then hands off to meerkatflow:spec.
---

# Brainstorm

A request usually carries less of the user's intent than it seems to. This skill closes that gap before any design exists: learn what they want and the reason behind it. Knowing the reason is what lets you settle the many small choices they will never mention in the way they would have settled them.

## 1. Decide whether to run at all

Ask yourself what you already know about the goal and its reason.

- The request already settles everything that matters (a one-line change such as "make the icon blue", or a handoff doc that fixes scope and checks) → do not run this skill. Continue with `meerkatflow:spec`, or edit directly if the session routing allows it.
- Something that would change the result is still open → continue below.

## 2. Do your homework first

Read whatever they referred to (files, spec, handoff, earlier messages) and the code around it, until you could explain how it works today and what the change would reach. Look in memory for earlier decisions. Do not ask anything the project or the conversation already answers.

When the change reshapes something that already runs (a screen, an endpoint), look at it running and at real data before proposing a design (`meerkatflow:measure`): a region or field the code can render may never have data, so count how often conditional parts actually appear.

## 3. Opening message

Two cases:

- **The request is already detailed** (a ticket, a long description). Restate it in your own words: what, for whom, and why, as far as they told you. Add what your homework turned up (current behavior, what the change reaches) and name anything that would alter the request or its implementation: a constraint, a conflict, a cheaper route. Keep it near 200 words. Finish with a line asking them to correct you, followed by a single question: the open point with the most influence on the result.
- **The request is short or vague.** Ask a single open question, in their words, about the largest open point, and say what its answer would decide.

If they did not ask for this step, add a line after the question, in the user's language, telling them they can ask you to go straight ahead. If they do, stop here: state any significant implementation choice in one plain line and continue with `meerkatflow:spec`.

## 4. Talking it through

- **A single question per message.** One open sentence without sample answers; their own phrasing tells you more than a choice between your guesses. Offer a short list only when they are stuck. When they ask for your guess, give a concrete proposal and the reason for it.
- **Follow the thread.** Let each question grow out of their previous reply or out of a finding, so it brings up an effect or a scenario they may have overlooked. If it is not obvious why you ask, say what the answer would change. Often a proposal they can correct works better than another question.
- **What you need to understand by the end** (a checklist for you, not questions to read out): what kind of thing this is; why it matters now and how success looks; goals, non-goals and anti-goals (outcomes that count as failure even when everything technically works); examples they like or dislike; how things stand today and what they already tried; the details they care about.
- **Who decides the implementation.** Early on, find out which choices (library, pattern, location in the code) they want to make themselves and which they leave to you. In an existing codebase, stay with what it already uses. Anything you decide stays marked as your decision.
- **Suggest more digging** when it would help (deeper code, logs, the web, the running app through `meerkatflow:measure`): say what you would look for and let them decide.
- Use their vocabulary and plain language. Do not describe this process to them.

## 5. Check your understanding

Each time one part is clear enough (who it is for, how one piece behaves), write it back in roughly 200 to 300 words, then move on to the next part. Present only what they said as theirs; anything you added is a proposal with a reason. Close every write-back by asking what is wrong or missing.

## 6. Ways to build it

When the goal is clear, send the options in a message of their own. Come up with several, discard the ones that work against the reason or that the existing code rules out, and present the remaining two or three with their trade-offs and change size (files, rough ± lines). Recommend one, say why, and ask them to choose. If only one way makes sense, tell them, with the reason each alternative fails.

Small implementation details that do not change the result (names, file locations, a default value) are not asked one at a time: collect them into one numbered list following the `meerkatflow:spec` question rules.

## 7. Handing over

Once they have chosen, summarise briefly: the intent and its reason; goals, non-goals and anti-goals; the chosen approach and the rejected ones with reasons; what they decided; what you decided. Then call `meerkatflow:spec` with that summary, and no other skill. If the work splits into independent parts, point that out, settle an order with them and run this skill for one part at a time.

## Limits

This skill produces understanding, not code. Reading, searching and measuring are allowed; writing code, scaffolding, installing packages and starting implementation skills are not. Agreement with an idea is not approval of a spec.
