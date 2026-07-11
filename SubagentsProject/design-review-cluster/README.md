# Design / UI-UX Review Cluster (Phase 2)

Three purpose-built subagents replacing a scattered set of design-review skills, most of which turned out to be empty template duplicates with no real content behind them. One orchestrator command fans the relevant ones out together.

## The three agents

| Agent | Mode | Covers |
|---|---|---|
| `ui-ux-reviewer` | Read-only, reports findings | Visual hierarchy, spacing, typography, color, alignment, layout, component/Figma hygiene, microcopy |
| `accessibility-reviewer` | Read-only, reports findings | WCAG 2.1 AA — perceivable/operable/understandable/robust, keyboard nav, screen readers, ARIA |
| `conversion-auditor` | Read-only, reports findings | Conversion friction quantified in dollars — cognitive load, forms, speed, trust, mobile, copy, checkout |

All three are read-only reviewers — none of them edit designs or copy directly, unlike Phase 1 where `code-simplifier` was a deliberate exception. Design/copy changes are creative decisions that belong with a human or a dedicated build tool, not an auto-applying critique agent.

## Why these three

**`ui-ux-reviewer`** is built on the FormFactor `design-review` skill's checklist — the only design-review asset in the whole install with genuine numeric, actionable criteria (16px minimum body text, 8-step spacing scale, 600px max text width, specific contrast rules). Microcopy quality was folded in as a checklist dimension rather than kept separate, since the `ux-writing-microcopy` skill it might have come from was a contentless template.

**`accessibility-reviewer`** is built on VoltAgent's `accessibility-tester`, rewritten into a tighter WCAG-principle structure (Perceivable/Operable/Understandable/Robust) with concrete checks instead of keyword lists. Kept separate from `ui-ux-reviewer` because accessibility is a compliance domain with its own named criteria (WCAG success criteria) — merging it in would bury pass/fail compliance facts inside subjective design taste.

**`conversion-auditor`** is built on `friction-auditor`, by far the richest single asset found in the whole design domain — real frameworks (Hick's/Miller's/Fitts's Law), dollar quantification, ROI ranking. Kept separate because it answers a fundamentally different question ("is this costing revenue") than the other two ("is this well-designed" / "is this compliant") — a page can be beautiful and accessible and still leak conversions, or ugly and still convert. The orchestrator explicitly skips this agent for non-funnel interfaces (internal tools, dashboards) since it has nothing meaningful to say there.

## What was mostly noise

Of the 12-skill a1e99 UI/UX pack added earlier, 11 shared a byte-for-byte identical template — same headings, same 5-step workflow, same output format — differing only by a one-line "Purpose" field, with no actual domain checklist inside. They looked like 11 distinct capabilities; they were one empty shell wearing 11 labels. This is the opposite of Phase 1's redundancy (there, agents genuinely duplicated real logic across plugins); here the "duplication" was mostly duplicated absence of content.

## Orchestrator

`commands/design-review-together.md` — fans the relevant agents out **in parallel** based on what's being reviewed (skips `conversion-auditor` for non-funnel interfaces), merges findings into one report (critical issues first regardless of source agent, then visual/accessibility/conversion), and merges same-element findings from multiple agents into one entry instead of listing them twice.

## Installing

```bash
mkdir -p ~/.claude/agents ~/.claude/commands
cp design-review-cluster/agents/*.md ~/.claude/agents/
cp design-review-cluster/commands/*.md ~/.claude/commands/
```

Invoke with `/design-review-together <screenshot/Figma URL/live URL>`, or call individual agents directly by name.

## Cleanup performed (2026-07-11)

Same litmus test as Phase 1: *did this cluster actually replace it, or is it still needed for something else?*

**Deleted** (empty-template duplicates or fully-superseded originals):
- a1e99 pack: `webapp-interface-review`, `dashboard-admin-ui-review`, `landing-page-ui-review`, `conversion-focused-ui-review`, `accessibility-review`, `ux-writing-microcopy`, `mobile-app-ui-design` — all shared the identical contentless template; their stated Purpose is now covered by `ui-ux-reviewer`'s checklist and `mobile-app-ui-design`'s own Purpose line ("Plan," not review) meant it wasn't really a review tool to begin with.
- `voltagent-qa-sec:accessibility-tester` — content fully carried into `accessibility-reviewer`.
- `friction-auditor` (original skill) — `conversion-auditor`'s prompt is a near-complete carry-forward of its content, not a summary; keeping both would be true duplication.

**Explicitly kept** (failed the litmus test — genuinely still useful separately):
- FormFactor's `design-review` skill — its `knowledge-base.md` (466 lines) is deeper than what got condensed into `ui-ux-reviewer`'s agent prompt, and it's a different invocation shape (a slow, thorough main-conversation skill vs. a fast fan-out subagent). Worth keeping both for their different use cases.

**Untouched entirely, out of scope for this cluster:** `design-system-builder`, `component-spec-writer`, `frontend-implementation-handoff`, `design-brief-builder`, `ui-designer`, `design-bridge`, `hero-strategy-director`/`hero-visual-director`/`navbar-strategy-director`/`scroll-engagement-designer`, `mobile-first-designer`/`desktop-focused-designer`, `cta-strategist`/`form-checkout-optimizer`, `landing-page-optimizer`, `sales-website-cro`, `ui-ux-tester` — see the analysis below for the mixed audit+build tools specifically.

## The mixed audit+build tools: phase candidate, or use-as-is?

Five skills didn't fit this cluster because they generate new work, not just critique existing work: `landing-page-optimizer`, `sales-website-cro`, `cta-strategist`, `form-checkout-optimizer`, `design-system-builder`. Same reasoning that kept `code-architect`/`code-explorer` out of Phase 1 — generative and critique are different jobs.

**Verdict: use situationally, don't make this a phase — with one exception worth watching.**

The whole point of this consolidation project is subagents that can *fan out together* for fast multi-angle review. That model doesn't transfer well to generative work: building a landing page is normally one creative task with one clear owner, not five angles converging on one answer. There's no natural "team" here the way there is for review.

That said, real redundancy exists in one spot: `landing-page-optimizer` and `sales-website-cro` both audit *and* build full landing/sales pages, and their build content overlaps (both generate headlines, CTAs, trust sections) — `landing-page-optimizer` is the more rigorous of the two (10-dimension CRO scoring rubric vs. general fundamentals). If these get used often enough to notice the overlap in practice, that's a two-into-one merge worth doing later — call it Phase 3 only if it actually comes up.

`cta-strategist` and `form-checkout-optimizer` are narrow, single-element builders (just the CTA, just the form) — small and focused enough that consolidating them wouldn't simplify anything; they're already about as lean as a skill gets. `design-system-builder` is a different job again (tokens/components, not page copy) with no overlap to merge against.

**Recommendation:** leave all five as-is, reach for them by name when the task is actually "build a new X" rather than "review this existing X" — that's what `conversion-auditor` is for. Revisit `landing-page-optimizer` vs. `sales-website-cro` specifically if you ever find yourself unsure which one to reach for.

## Status

Phase 2 of the subagent consolidation project — see `agent-consolidation-project.md` in memory for the full plan.
