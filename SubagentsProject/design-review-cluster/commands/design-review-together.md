---
description: Fans out ui-ux-reviewer, accessibility-reviewer, and conversion-auditor together for a fast multi-angle review of a screenshot, Figma file, or live page/URL.
argument-hint: "[screenshot path, Figma URL, live URL, or description of what to review]"
---

This is the Phase 2 design/UI-UX review cluster: three purpose-built agents covering visual design quality (`ui-ux-reviewer`), accessibility compliance (`accessibility-reviewer`), and conversion friction (`conversion-auditor`) — run together instead of one at a time.

Target: `$ARGUMENTS`

1. Determine what's being reviewed:
   - If `$ARGUMENTS` is a file path to a screenshot/image: use that directly.
   - If it's a Figma URL or live URL: use that directly.
   - If it's empty or a plain description: ask what to review (a screenshot, a Figma link, or a live URL) — none of these three agents can produce a meaningful review from nothing.

2. Not every review needs all three agents. Ask yourself, and ask the user if genuinely unclear:
   - Reviewing a marketing/landing page or anything meant to convert visitors? Run all three.
   - Reviewing an internal tool, dashboard, or app screen with no conversion goal? Run `ui-ux-reviewer` and `accessibility-reviewer`, skip `conversion-auditor` — it will have nothing meaningful to say about a page with no funnel.
   - Reviewing pure visual polish with accessibility already handled elsewhere? `ui-ux-reviewer` alone is fine — don't force the others in.

3. Launch the selected agents **in parallel** (single message, multiple Agent tool calls) with self-contained prompts that each include: the target (file path/URL/screenshot), and for `conversion-auditor` specifically, current conversion rate and traffic if the user has provided them (otherwise tell it to mark impact estimates as illustrative).

4. Synthesize the findings into one report, in this order:
   - **Critical issues** across all agents first (regardless of which agent found them) — anything that blocks a user or clearly costs revenue
   - **Visual design** (from `ui-ux-reviewer`)
   - **Accessibility** (from `accessibility-reviewer`, grouped by WCAG principle)
   - **Conversion friction** (from `conversion-auditor`, if run) — lead with the #1 conversion killer if one was named
   - If two agents flag the same element from different angles (e.g. low-contrast CTA is both a visual-hierarchy issue and a WCAG contrast violation), merge into one entry noting both angles rather than listing it twice.

5. If nothing of substance was found, say so plainly — don't pad the response to look thorough.
