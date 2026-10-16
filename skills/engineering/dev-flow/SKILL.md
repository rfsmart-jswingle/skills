---
name: dev-flow
description: Walk through the development flow one step at a time. Works out where the current work is, names the next skill to run, and says whether to continue, clear, or compact first.
disable-model-invocation: true
---

# Dev Flow

Guide the user through the flow from idea to pull request, one step per invocation. The user runs `/dev-flow` at each phase boundary; you work out where they are and tell them what comes next.

Before stating what a skill does or recommending a step be skipped, read that skill's SKILL.md: the summaries here are for orientation only.

Repo-specific steps (branch naming, ticket rules, lint and test commands, PR and review process) live in the repo's own docs (`AGENTS.md`/`CLAUDE.md`). Fit the flow below around them; where they conflict, the repo wins.

## Each invocation

1. **Read the current state from the repo.** Keep no state file: the repo is always current, and a state file goes stale the moment someone works outside this skill. Check:
   - the current branch, and any ticket reference in its name;
   - that ticket in the issue tracker, and its status;
   - a spec file for this work in the repo's spec folder, and tickets linking to it;
   - the diff against the default branch, and uncommitted changes;
   - an open pull request for the branch.

   If the state doesn't show which entry path this is, ask (see Entry paths).

2. **Name the phase and the next step.** Say which phase the work is in, which skill to run next, and why. User-invoked skills are run by the user: give them the command. Model-invoked skills you may call yourself.

3. **Name the boundary move.** Say whether to continue in this session, `/clear`, `/handoff`, use a subagent, or `/compact` before the next step (see Phase boundaries).

Keep the answer short: phase, next step, boundary move, one line of reasoning each.

## Entry paths

Three ways work starts. Each joins the main flow.

| Starting point | Path |
|---|---|
| **Your own idea** | `/grill-with-docs` → main flow |
| **A ticket someone else wrote** | `/triage` → gaps filled only if needed → `/implement` |
| **A raw incoming issue** (bug report, request) | `/triage` → it routes to one of the paths here, `diagnosing-bugs`, or `/wayfinder` |

For a ticket someone else wrote, triage checks the work isn't already built, reproduces it if it's a bug, and lists the gaps. With no gaps it goes straight to `/implement`. With gaps it runs a light grilling, research, or prototype aimed only at them and writes the answers back into the ticket, so the ticket stays the spec.

## The main flow: idea → pull request

1. **`/grill-with-docs`** sharpens the idea by interview, recording terms in the glossary and hard-to-reverse decisions as DRs (decision records) as they land.
2. **Detour, only if a question needs a runnable answer** (state, business logic, a UI you have to see): `/handoff` out, open a fresh session against that file, run `prototype`, then `/handoff` back what you learned. Facts from outside the repo go to `research` instead, which runs as a background agent. Both are detours inside the grilling, not stages after it.
3. **Branch: is this a multi-session build?**
   - **Yes** → **`/to-spec`** writes the spec to a dated file in the repo, settling the test seams with the user. Then **`/to-tickets`** splits it into tracer-bullet tickets with blocking edges, in the repo's ticket conventions. Then **`/implement`** one ticket at a time, blockers first, **`/clear`ing between tickets**. Each ticket links the spec, so the last ticket's context is disposable.
   - **No** → **`/implement`** right here, in the same context window.
4. **`/implement`** loads `codebase-design`, builds test-first with `tdd` at the agreed seams, runs the repo's lint, typecheck, and test commands, closes with `code-review` (Standards + Spec), and commits.
5. **Pull request.** Call the Skill tool with "pr" to write the body. Then open the PR and handle review using the repo's conventions. This is where the flow ends.
6. **`/retro`**, optional. After a build that went sideways, it looks back over the session and suggests changes to the agent's environment (pointers, automated checks, coding standards), not the code. Run it in the session it's looking back on, before you clear.

### Context hygiene

Keep steps 1–3 in **one unbroken context window** (don't compact or clear until after `/to-tickets`) so the grilling, spec, and tickets all build on the same thinking. Each `/implement` then starts fresh, working from its ticket.

The limit on this is the **[smart zone](https://www.aihero.dev/ai-coding-dictionary/smart-zone)**: the window (~150k tokens on state-of-the-art models) within which the model still reasons sharply. If a session approaches it before `/to-tickets`, don't push on degraded; `/compact` at the nearest phase boundary and carry on.

## Routes off the main flow

- **Something's broken** → `diagnosing-bugs`. For the hard ones: the bug that resists a first glance, the intermittent flake, the regression between two known-good states. It builds a feedback loop that goes red on *this* bug before theorising, then fixes with a regression test. If the real finding is that there's no good seam to lock the bug down, that's a job for `/improve-codebase-architecture`.
- **A huge, foggy effort, too big for one session** → `/wayfinder`. It charts a map of decision tickets and resolves them one per session, producing decisions, not deliverables. When the map clears, it joins the main flow at `/to-spec`. Save it for exactly that, never a well-scoped feature.
- **Codebase upkeep** → `/improve-codebase-architecture` surveys for deepening opportunities. Picking one generates an idea that enters the main flow at `/grill-with-docs`.

## Vocabulary underneath

Two model-invoked references that the skills above pull in. Reach for them directly when the **words**, not the process, are the problem.

- **`domain-modeling`**: the project's domain language. Challenge a fuzzy term, resolve an overloaded word, record a hard-to-reverse decision as a DR.
- **`codebase-design`**: the deep-module vocabulary (module, interface, depth, seam, adapter, leverage, locality) for designing a module's shape. `/to-spec`, `/implement`, `tdd`, and `/improve-codebase-architecture` load it.

## Phase boundaries

A **phase** is a chunk of work inside a session: the grilling, the implementation, the QA. At the **boundary** between two of them there are five options:

- **Continue**: stay put. Costs nothing, loses nothing.
- **`/clear`**: empty the window, when nothing here matters to what's next.
- **`/handoff`** writes a portable markdown file. Narrow: only for a **new harness**, a **new directory**, a **colleague**, or forking a side task **mid-phase**.
- **Subagent**: send a tightly-scoped task to its own window and get a report back.
- **`/compact`** compresses this context and seeds a fresh session with it. The **default**, at the bottom of the tree rather than the first reach.

Read [PHASE-BOUNDARIES.md](PHASE-BOUNDARIES.md) for the ordered tree and the reasoning behind each branch. Make the decision **at** a boundary; mid-phase, continue or split the rest into subagents.

## Standalone

Off the flow entirely, for when the situation calls for them.

- **`/grill-me`**: the same interview as `/grill-with-docs`, but saves nothing. Use it when there is no repo under the work.
- **`grilling`**: the interview primitive itself, run inside `/grill-me`, `/grill-with-docs`, `/triage`, `/wayfinder`, and `/improve-codebase-architecture`.
- **`/to-questionnaire`**: when the blocker is in someone else's head, write them a questionnaire.
- **`wizard`**: an interactive script for steps only a human can take (credentials, dashboards, one-off cutovers).
- **`/wait-what`**: re-pitch a message that didn't land, in plain words and glossary terms.
- **`/teach`**: learn a concept over several sessions.
- **`writing-for-agents`**: the reference for writing skills, `AGENTS.md`, and other docs agents read.
