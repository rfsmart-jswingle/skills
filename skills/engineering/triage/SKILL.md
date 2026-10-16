---
name: triage
description: Evaluate an incoming issue, or a ticket someone else wrote that you're picking up, then verify it, fill any gaps, and route it to the right next step.
disable-model-invocation: true
---

# Triage

Work out what an item needs before anyone builds it, and route it to the next step. Triage is mainly a **router**: it settles whether the work is real, new, and clear enough, then names the path it should take.

Two kinds of input:

- **An incoming issue**: a bug report or request someone raised, from any source. Every issue is treated the same, wherever it came from.
- **A ticket someone else wrote** that you are picking up to build. Same checks, but the goal is to find the gaps before implementing, not to hand it on.

Read and write the issue tracker using the repo's documented conventions (check `AGENTS.md`/`CLAUDE.md`). Don't invent labels, statuses, or comment formats the repo doesn't already use.

## Invocation

The user invokes `/triage` and describes what they want in natural language. Interpret the request and act. Examples:

- "Show me anything that needs my attention"
- "Let's look at #42"
- "I'm picking up PROJ-123"

## Show what needs attention

Query the tracker for items nobody has evaluated yet, plus items where the reporter has replied since the last triage notes. Use the repo's conventions to tell which is which. Show a count and a one-line summary per item, oldest first. Let the user pick.

## Triage one item

1. **Gather context.** Read the full item: body, comments, author, dates. Parse any prior triage notes so you don't re-ask resolved questions. Explore the codebase using the project's domain glossary, respecting DRs (decision records) in the area. Then check for **redundancy**: search for an existing implementation of the requested behavior by domain concept (not just the request's wording), and report where you looked. Check the repo's DRs too: a request may already have been deliberately turned down.

2. **Verify the claim.** Before any grilling, check that the claim holds up. For a bug, reproduce it from the reporter's steps. Report what happened: confirmed (with code path), failed, or insufficient detail.

3. **Find the gaps.** List what is still unclear: missing behavior, open design questions, unknown facts. Tell the user your findings so far and your recommended outcome, and wait for direction.

4. **Fill the gaps (only if needed).** Keep it light and aimed only at the gaps:
   - A decision the user can make: call the Skill tool twice, for "grilling" and "domain-modeling".
   - A fact outside the repo: call the Skill tool with "research".
   - A question that needs something runnable to answer: call the Skill tool with "prototype".

   For a ticket you're picking up, write the answers back into the ticket so it stays the spec.

5. **Apply the outcome.** Exactly one of:
   - **Already built**: point to where it lives, and close the item per the repo's conventions.
   - **Won't do**: give a polite reason and close. If the reason will still hold next time someone asks, offer to record it as a DR so the request isn't re-proposed.
   - **Needs info**: post triage notes (template below) and leave it open.
   - **Ready**: post your findings on the item and name the next path (see below). Don't create tickets here: the path you route to creates them at the right point.

## Routing a ready item

Recommend one path. The user starts it; triage doesn't run it.

- **Small and clear, one session**: `/implement`. This is the usual path for a ticket you're picking up once its gaps are filled.
- **Clear but needs design or a spec**: `/grill-with-docs`, then the main flow (see `/dev-flow`).
- **A hard bug**: the one that resists a first glance, an intermittent flake, a regression. Call `diagnosing-bugs`.
- **Huge or foggy, more than one session can hold**: `/wayfinder`.

## Needs-info template

```markdown
## Triage Notes

**What we've established so far:**

- point 1
- point 2

**What we still need from you (@reporter):**

- question 1
- question 2
```

Capture everything resolved during grilling under "established so far" so the work isn't lost. Questions must be specific and actionable, not "please provide more info".

## Resuming a previous session

If prior triage notes exist on the item, read them, check whether the reporter has answered any outstanding questions, and present an updated picture before continuing. Don't re-ask resolved questions.
