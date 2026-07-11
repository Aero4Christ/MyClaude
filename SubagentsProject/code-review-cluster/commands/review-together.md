---
description: Fans out code-reviewer and architecture-critic together for a fast multi-angle review of a diff/path, then offers to run code-simplifier on anything left over once findings are addressed.
argument-hint: "[optional path, or 'whole app'/'whole repo' for a full pass]"
---

This is the Phase 1 code/architecture review cluster: three purpose-built agents that cover correctness+security+error-handling+tests (`code-reviewer`), structural/design quality (`architecture-critic`), and clarity cleanup (`code-simplifier`) — run together instead of one at a time.

Scope: `$ARGUMENTS`

1. Determine what to hand the agents:
   - If `$ARGUMENTS` is empty: scope to the current diff. Run `git status --short` and `git diff HEAD`, including untracked new source files (read them directly). If there's no uncommitted diff, fall back to the most recent commit (`git diff HEAD~1`) or ask what to review.
   - If `$ARGUMENTS` names a path: scope to that path.
   - If `$ARGUMENTS` says "whole app"/"whole repo"/similar: scope to the entire current project.

2. Launch `code-reviewer` and `architecture-critic` **in parallel** (single message, two Agent tool calls) with self-contained prompts that each include: the repo root, the scope from step 1 (actual files/diff listed, not "figure it out"), and a reminder to read files directly rather than relying only on the diff text. Do not launch `code-simplifier` yet — it edits files, and shouldn't run until the other two have reported and any real findings are addressed, otherwise it may "clean up" code that's about to be rewritten anyway.

3. Synthesize both agents' findings into one report, in this order:
   - **Correctness & Security** (from `code-reviewer`, Critical then Important)
   - **Error-Handling & Test Gaps** (from `code-reviewer`)
   - **Architecture** (from `architecture-critic`, Blocker then High then Medium/Nit)
   - Don't just concatenate — if both agents flag the same location, merge into one entry noting both angles.

4. If findings exist, ask whether to fix them now, and whether to also run `code-simplifier` afterward as a cleanup pass over whatever ends up changed. If both agents find nothing of substance, say so plainly — don't pad the response, and don't run the simplifier unprompted (its own SKILL note: it should not go rewriting untouched code on its own initiative).

5. If the user opts into simplification, launch `code-simplifier` scoped to the same diff/path, relay its summary of what changed, and stop — it applies its own edits directly, no further agent needed.
