---
description: Fans out unreal-code-reviewer and unreal-architecture-critic together for a fast multi-angle review of Unreal Engine gameplay code (diff/path) — engine-specific correctness (UObject lifetime, replication, threading, GAS) plus structural/design quality.
argument-hint: "[optional path, or 'whole project' for a full pass]"
---

This is the Phase 3 Unreal/game-dev review cluster: two purpose-built agents that cover UE-specific correctness + general correctness/security/error-handling/tests (`unreal-code-reviewer`) and structural/design quality (`unreal-architecture-critic`) — run together instead of one at a time. This is additive to the existing `ue-*` build-time skills (GAS, networking, animation, etc.) — those still auto-trigger while you write code; this command is for reviewing what got written.

Scope: `$ARGUMENTS`

1. Determine what to hand the agents:
   - If `$ARGUMENTS` is empty: scope to the current diff. Run `git status --short` and `git diff HEAD`, including untracked new source files (read them directly). If there's no uncommitted diff, fall back to the most recent commit (`git diff HEAD~1`) or ask what to review.
   - If `$ARGUMENTS` names a path: scope to that path.
   - If `$ARGUMENTS` says "whole project"/"whole repo"/similar: scope to the entire current Unreal project (Source/ and any Blueprint-relevant C++ glue).

2. Check whether `.agents/ue-project-context.md` exists in the project. If not, mention it once in your summary and suggest running the `ue-project-context` skill — the review agents will still run with sane defaults, but engine-version- and convention-specific findings will be less precise without it.

3. Launch `unreal-code-reviewer` and `unreal-architecture-critic` **in parallel** (single message, two Agent tool calls) with self-contained prompts that each include: the repo root, the scope from step 1 (actual files/diff listed, not "figure it out"), and a reminder to read files directly rather than relying only on the diff text.

4. Synthesize both agents' findings into one report, in this order:
   - **UObject Lifetime, Replication, Threading, GAS** (from `unreal-code-reviewer`, Critical then Important)
   - **General Correctness, Security, Error-Handling & Test Gaps** (from `unreal-code-reviewer`)
   - **Architecture** (from `unreal-architecture-critic`, Blocker then High then Medium/Nit)
   - Don't just concatenate — if both agents flag the same location, merge into one entry noting both angles.

5. If findings exist, ask whether to fix them now. If both agents find nothing of substance, say so plainly — don't pad the response. Unlike the general code-review cluster, there is no Unreal-specific `code-simplifier` yet — if cleanup is wanted after fixes land, note that the general `code-simplifier` agent can be used, since UE C++ is still C++.
