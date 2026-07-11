# Code & Architecture Review Cluster (Phase 1)

Three purpose-built subagents that replace a scattered set of overlapping review agents from four different plugins (`pr-review-toolkit`, `code-modernization`, `feature-dev`, `voltagent-qa-sec`, `code-simplifier`). One orchestrator command fans the relevant ones out together.

## The three agents

| Agent | Mode | Covers |
|---|---|---|
| `code-reviewer` | Read-only, reports findings | Correctness, security (OWASP/CWE/secrets), silent failures/error handling, test-coverage gaps |
| `architecture-critic` | Read-only, reports findings | Structural/design quality, over-engineering, simpler alternatives |
| `code-simplifier` | **Mutates files** | Clarity/consistency cleanup on recently changed code — quality only, no bug hunting |

## Why these three, and not more or fewer

**Merged into `code-reviewer`:** correctness bug-hunting, security scanning, silent-failure/error-handling auditing, and test-coverage gap analysis. All four are read-only passes over the same diff that just *report* what they find — no functional conflict in combining them, and running one agent instead of four means one fast pass instead of four round-trips for things that are naturally checked together anyway.

**Kept separate: `architecture-critic`.** Structural/design judgment operates at a different altitude than line-level bug hunting — it needs to read broader context than the diff (surrounding modules, service boundaries) and asks a categorically different question ("is this the right shape?" vs "is this line wrong?"). Folding it into `code-reviewer` would blur two jobs that benefit from staying distinct.

**Kept separate: `code-simplifier`.** It's the only agent in the cluster with Write/Edit access — it mutates code, the other two only report. Mixing report-only and auto-edit behavior in one agent would muddy the risk profile: you want to see what the other two found *before* anything starts rewriting files, not have a single agent both critique and edit in the same breath.

## What got retired as standalone agents

These aren't gone — their concerns live on as checklist dimensions inside `code-reviewer`, rather than separate agent invocations:

- `pr-review-toolkit:silent-failure-hunter` → error-handling dimension
- `pr-review-toolkit:pr-test-analyzer` → test-coverage dimension
- `pr-review-toolkit:type-design-analyzer` → folded into correctness (not a dedicated dimension — this agent's file is also kept installed for a deep type-design pass, see below)
- `code-modernization:security-auditor` → security dimension (the code-vuln-focused one; its broader systems/compliance sibling in `voltagent-qa-sec` is out of scope for this cluster, and is also kept installed, see below)
- Four near-duplicate `code-reviewer` agents (`feature-dev`, `pr-review-toolkit`, `voltagent-qa-sec`, built-in `/code-review` skill) → collapsed into this one
- Two duplicate `code-simplifier` agents (standalone plugin + `pr-review-toolkit` copy, word-for-word identical) → collapsed into this one
- `code-modernization:architecture-critic` → carried forward almost unchanged, generalized from "modernization-specific" to general-purpose

## Cleanup performed (2026-07-10)

Once this cluster was working, the superseded originals were deleted so they don't sit around as dead weight. Litmus test used throughout: *"did this cluster actually replace it, or is it still needed for something else?"*

**Deleted** (fully replaced, confirmed no other feature depends on them):
- `pr-review-toolkit`: `code-reviewer.md`, `code-simplifier.md`, `silent-failure-hunter.md`, `pr-test-analyzer.md`, `commands/review-pr.md` (its orchestration job is superseded by `review-together`)
- `feature-dev/agents/code-reviewer.md` (kept `code-architect.md`/`code-explorer.md`/the `feature-dev` command — a build workflow, never touched by this cluster)
- `voltagent-qa-sec/code-reviewer.md`
- Standalone `code-simplifier` plugin — uninstalled entirely (`claude plugin uninstall code-simplifier`); its only content was the agent this cluster replaced
- The old thin wrapper command `~/.claude/commands/architecture-review.md` — superseded by `review-together` + the new `architecture-critic`

**Explicitly kept, not deleted** (failed the litmus test — genuinely not replaced):
- `pr-review-toolkit:comment-analyzer` — comment/docstring-rot detection was deliberately *not* folded into `code-reviewer` (lower signal); nothing else covers it
- `pr-review-toolkit:type-design-analyzer` — only partially folded; still useful standalone for a deep type-encapsulation pass
- `voltagent-qa-sec:security-auditor` — broader infra/compliance/systems-level scope than the code-vuln-focused security dimension folded into `code-reviewer`; genuinely different job

**Untouched entirely, out of scope:**
- `code-modernization` plugin, in full — its `architecture-critic.md` and `security-auditor.md` look like duplicates of ours but are load-bearing for that plugin's own `/modernize-uplift`, `/modernize-transform`, `/modernize-reimagine`, `/modernize-harden`, `/modernize-assess` pipeline. Deleting them would have broken a separate feature this project isn't replacing.
- Built-in `/code-review` and `/simplify` skills — shipped inside the Claude Code binary itself, not files, can't be deleted. They still functionally overlap with `review-together`; just use `review-together` instead of reaching for these.
- Every `voltagent-qa-sec` agent not named above (`architect-reviewer`, `debugger`, `error-detective`, `qa-expert`, `test-automator`, `chaos-engineer`, `accessibility-tester`, `ui-ux-tester`, `compliance-auditor`, `gdpr-ccpa-compliance`, `ad-security-reviewer`, `powershell-security-hardening`, `penetration-tester`, `ai-writing-auditor`, `performance-engineer`) — none of these were folded into this cluster, so none were touched.

## Orchestrator

`commands/review-together.md` — fans `code-reviewer` and `architecture-critic` out **in parallel**, merges their findings into one ranked report (correctness/security first, then error-handling/tests, then architecture), and only offers to run `code-simplifier` afterward once real findings are addressed. This is "one review, three angles" instead of three separate manual invocations.

## Installing

These are plain Claude Code agent/command markdown files. To use them:

```bash
mkdir -p ~/.claude/agents ~/.claude/commands
cp code-review-cluster/agents/*.md ~/.claude/agents/
cp code-review-cluster/commands/*.md ~/.claude/commands/
```

Then invoke with `/review-together` (optionally with a path or "whole repo"), or call the individual agents directly via the Agent/Task tool by name.

## Status

Phase 1 of a larger project to replace most of a ~350-skill install with small, domain-segmented subagent clusters that can be fanned out together. Phase 2 (design/UI-UX review cluster) is next — see the project's memory note `agent-consolidation-project.md` for the full plan.
