---
name: implement
description: "Implement a piece of work based on a spec or set of tickets."
disable-model-invocation: true
---

Implement the work described by the user in the spec or tickets.

If the user passes a ticket reference, fetch it from the issue tracker and state its title before starting. If the reference is ambiguous, ask. If the ticket links a spec, read it.

If the repo requires a ticket for every change (check `AGENTS.md`/`CLAUDE.md`) and there isn't one yet, create it first using the repo's conventions. Follow the repo's branch conventions too.

Before writing code, call the Skill tool with "codebase-design" for the vocabulary of modules, interfaces, depth, and seams.

Call the Skill tool with "tdd" where possible, at pre-agreed seams.

Use whatever lint, typecheck, and test commands the repo has. Run typechecking and single test files regularly, and the full test suite once at the end.

Once done, call the Skill tool with "code-review" to review the work.

Commit your work to the current branch, following the repo's commit conventions.
