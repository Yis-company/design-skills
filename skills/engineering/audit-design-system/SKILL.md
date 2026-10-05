---
name: audit-design-system
description: Scan a codebase for drift from its design system, present it as a visual HTML report, then grill through whichever finding you pick.
disable-model-invocation: true
---

# Audit Design System

Surface **drift** between the code and the design system, and propose how to fold it back in. The aim is consistency for users and fewer one-offs to maintain.

This command is _informed_ by the project's domain model and built on a shared design-system vocabulary:

- Call the Skill tool with "design-system" for the vocabulary (**token** and its primitive, semantic and component tiers, **component**, **variant**, **slot**, **drift**, **one-off**, **source of truth**) and its principles (semantic tokens at use sites, the deletion test, "one use = hypothetical variant, two = real", "the component API is the design contract"). Use these terms exactly in every suggestion.
- The domain language in `GLOSSARY.md` names the screens and concepts; ADRs in `docs/adr/` record decisions this command should not re-litigate. If `docs/agents/design-source.md` exists, read it for where the design source, token files and component library live.

Read-only on production code: the run produces a conversation and one HTML file outside the repo. Never call a design-tool MCP; if a design-source link can't be opened, ask the user for exports.

## Process

### 1. Explore

**Scope before you scan: YAGNI.** Folding drift back in pays off where the UI keeps changing, so put extra weight on the parts of the codebase that have recently changed. Decide *where* to look before you look:

- If the user named a direction (a screen, a component family, a token group), take it, and skip the inference below.
- Otherwise, walk back a good stretch of the commit history (`git log --oneline`) to find the UI hot spots, the screens and components that keep coming up, and let those paths pull your attention first. If the changes are scattered with no clear hot spot, widen the net.

Read the project's domain glossary (`GLOSSARY.md`) and any ADRs in the area you're touching first.

Then spawn a sub-agent to walk the token files, the component library, and the screens in scope. Explore organically and note where the system and the code part ways:

- Hardcoded values (colour, spacing, type, radius, shadow, duration) where a token exists.
- Near-duplicate components: two or more that differ by a prop or a style tweak.
- One-off variants: style overrides or single-use props standing in for a variant.
- Unused tokens: defined, never read.
- Components whose API (props, variants, slots) diverges from the design-source component.

Apply the **deletion test** to anything you suspect is a near-duplicate: would deleting it and using the other change the screens meaningfully, or just remove a copy? Apply "one use = hypothetical variant" before proposing any new variant or token.

### 2. Present candidates as an HTML report

Write a self-contained HTML file to the OS temp directory so nothing lands in the repo. Resolve the temp dir from `$TMPDIR`, falling back to `/tmp` (or `%TEMP%` on Windows), and write to `<tmpdir>/design-system-audit-<timestamp>.html` so each run gets a fresh file. Open it for the user (`xdg-open <path>` on Linux, `open <path>` on macOS, `start <path>` on Windows) and tell them the absolute path.

For each candidate, render a card with:

- **Files**: which files and components are involved
- **Drift**: what has parted from the system, and how many use sites it touches
- **Proposal**: plain English description of what would change (use token X, merge into component Y, add variant Z, delete token W)
- **Benefit**: in terms of consistency and fewer one-offs
- **Before / After visual**: side-by-side, rendered from the real values: colour swatches, a rendered type scale, spacing bars, or component variants next to each other
- **Recommendation strength**: one of `Strong`, `Worth exploring`, `Speculative`, rendered as a badge

End the report with a **Top recommendation** section: which candidate you'd tackle first and why.

**Use GLOSSARY.md vocabulary for the domain, and the design-system vocabulary for the system.** If `GLOSSARY.md` defines "Checkout", talk about "the Checkout summary card", not "the `SummaryBoxV2`".

**ADR conflicts**: if a candidate contradicts an existing ADR, only surface it when the drift is real enough to warrant revisiting the ADR. Mark it clearly in the card (e.g. a warning callout: _"contradicts ADR-0007, but worth reopening because…"_). Don't list every theoretical cleanup an ADR forbids.

See [HTML-REPORT.md](HTML-REPORT.md) for the full HTML scaffold, visual patterns, and styling guidance.

Do NOT design the fix yet. After the file is written, ask the user: "Which of these would you like to explore?"

### 3. Grilling loop

Once the user picks a candidate, call the Skill tool twice, for "grilling" and "domain-modeling". Grilling walks the decision tree with them: which side is the source of truth, which tier the token belongs in, whether a second use justifies the variant, how use sites migrate. Domain-modeling keeps the domain model current as decisions crystallize:

- **Naming a token, component or variant after a concept not in `GLOSSARY.md`?** Add the term to `GLOSSARY.md`. Create the file lazily if it doesn't exist.
- **Sharpening a fuzzy term during the conversation?** Update `GLOSSARY.md` right there.
- **User rejects the candidate with a load-bearing reason?** Offer an ADR, framed as: _"Want me to record this as an ADR so future audits don't re-suggest it?"_ Only offer when the reason would actually be needed by a future auditor; skip ephemeral reasons ("not worth it right now") and self-evident ones.

The audit ends with a decision, not a diff. Tell the user to take it to `/to-spec` (a change engineers need specified) or `/to-tickets` (a token migration or a set of fixes ready to slice), or record it as an ADR.
