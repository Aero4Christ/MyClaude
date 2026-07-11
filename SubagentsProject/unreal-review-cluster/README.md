# Unreal/Game-Dev Review Cluster (Phase 3)

Two purpose-built subagents that add a review layer on top of Lightbound's existing UE build-time skill pack. This cluster is **purely additive** — nothing was deleted, edited, or restructured from the 27 `ue-*` skills already installed in `~/.agents/skills` (symlinked into `~/.claude/skills`).

## Why this cluster is shaped differently than Phase 1/2

Phase 1 (code review) and Phase 2 (design/UI-UX review) both merged genuinely duplicate read-only agents that were re-scanning the same diff. The `ue-*` pack isn't that: it's third-party reference knowledge (from `quodsoler/unreal-engine-skills`) where each skill covers a distinct, non-overlapping engine subsystem (GAS, replication, animation, Niagara, etc.) and auto-triggers by description match while you write code. There's nothing redundant in that pack to consolidate without losing coverage, and it's actively maintained upstream and load-bearing for the Lightbound build workflow — so it was left untouched.

What was actually missing was a **review** pass, not a **build-time knowledge** pass — the same gap Phase 1 filled for general code. So Phase 3 adds two new review agents in the same shape as `code-reviewer`/`architecture-critic`, scoped to Unreal-specific concerns.

## The two agents

| Agent | Mode | Covers |
|---|---|---|
| `unreal-code-reviewer` | Read-only, reports findings | UObject lifetime/GC safety, replication correctness, async/threading safety, GAS misuse, plus the general correctness/security/error-handling/test-gap dimensions |
| `unreal-architecture-critic` | Read-only, reports findings | Actor/Component/Subsystem design smells, GAS trio overuse, module boundaries, replication/rollout story — structural judgment, not line-level bugs |

Both read `.agents/ue-project-context.md` for engine version and conventions, same as the existing `ue-*` skills, but degrade gracefully with sane UE5 defaults if it's absent.

## What was NOT touched

- All 27 `ue-*` skills in `~/.agents/skills` / `~/.claude/skills` — zero edits, zero deletions
- `ue-project-context` skill and its generated `.agents/ue-project-context.md` files
- The VibeUE MCP setup used to drive Unreal Engine directly from Claude
- No files from `~/.agents/skills` or the `quodsoler/unreal-engine-skills` upstream sync were copied, forked, or modified

## Orchestrator

`commands/unreal-review-together.md` — fans `unreal-code-reviewer` and `unreal-architecture-critic` out **in parallel**, checks for `.agents/ue-project-context.md` and flags if it's missing, then merges findings into one ranked report (UE-specific critical issues first, then general correctness/security/tests, then architecture).

There is no Unreal-specific `code-simplifier` yet — the general-purpose `code-simplifier` agent works fine as a cleanup pass since UE C++ is still C++; a dedicated one can be added later if UE-specific cleanup patterns (e.g. macro/reflection idioms) turn out to need it.

## Installing

```bash
mkdir -p ~/.claude/agents ~/.claude/commands
cp unreal-review-cluster/agents/*.md ~/.claude/agents/
cp unreal-review-cluster/commands/*.md ~/.claude/commands/
```

Then invoke with `/unreal-review-together` (optionally with a path or "whole project"), or call the individual agents directly via the Agent/Task tool by name.

## Status

Phase 3 of the larger project to replace most of a ~350-skill install with small, domain-segmented subagent clusters that can be fanned out together. See the project's memory note `agent-consolidation-project.md` for the full plan and Phase 1/2 history.
