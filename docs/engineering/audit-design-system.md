## What it does

`audit-design-system` surveys a codebase for **drift**: places where what ships has parted from the design system. That covers hardcoded values where a token exists, near-duplicate components, one-off variants, unused tokens, and components whose code API no longer matches the design-source component. It writes them up as a self-contained HTML report with a visual before/after for each, and then [grills](https://www.aihero.dev/ai-coding-dictionary/grilling) you through the one you pick.

It never changes the code. The whole run produces a conversation and one HTML file in your OS temp directory. The fix happens later, through a spec or tickets that engineers pick up. This makes it a survey, not a cleanup tool, so you can run it on a codebase you have no plans to touch yet.

Two filters stop the report from becoming a list of every pixel that differs. First, a proposed new variant or token needs two uses: one use stays a one-off until a second shows up. Second, unless you point it at a specific area, it reads recent commit history first and focuses the scan on the screens and components that change often. Folding drift back in where nobody works is a cleanup that never pays back.

## When to reach for it

You invoke this by typing `/audit-design-system`; the [agent](https://www.aihero.dev/ai-coding-dictionary/agent) will not reach for it on its own.

It is not a step in the main designer flow. You run it periodically to queue up work that brings the code back in line with the system. People use it in three situations:

| Situation | How it is used |
| --- | --- |
| Routine upkeep | Run it every few weeks, or after a busy release, so drift does not pile up between features. |
| Before a big feature | Point it at the [spec](https://www.aihero.dev/ai-coding-dictionary/spec) and ask which components and tokens the feature touches have drifted, so the system gets fixed first. |
| Inheriting a codebase | Run it on a product you have just started designing for, to find out how far the code and the design source have parted. |

Where it is confusable with siblings:

- To check one branch or PR against its design, use [design-qa](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/design-qa.md). This skill looks at the whole codebase; `design-qa` looks at one diff.
- To settle what a design-system word means, use [design-system](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/design-system.md). This skill uses that vocabulary; it does not define it.

## Prerequisites

None to run it. If `GLOSSARY.md` or ADRs in `docs/adr/` exist, it reads them and uses your product's own nouns, so a candidate reads as "the Checkout summary card," not "`SummaryBoxV2`." If `docs/agents/design-source.md` exists (written by [setup-design-skills](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/setup-design-skills.md)), it uses it to find the design source, token files and component library.

It writes in two places. The report goes to `<tmpdir>/design-system-audit-<timestamp>.html`, outside the repo. During the grilling loop it adds or sharpens terms in `GLOSSARY.md`, and creates that file if it does not exist. It also offers to record a rejected candidate as an ADR, so a future run does not suggest it again.

## Drift, and the report that shows it

The skill rests on one idea: **drift**, any gap between the design system and what ships, measured against the source of truth. Each candidate is a card with the files involved, the drift, a plain-English proposal, the benefit in terms of consistency and fewer one-offs, a strength badge, and a before/after drawn from the real values: colour swatches, a rendered type scale, spacing bars, or component variants side by side.

| Badge | What it means for you |
| --- | --- |
| `Strong` | Clear drift across several use sites, with a token or component already there to fold it into. Take these seriously. |
| `Worth exploring` | Real drift, but the fix depends on a decision: which side is the source of truth, or whether a variant is earned. |
| `Speculative` | Included for completeness. You can ignore most of these. |

The report ends with a **Top recommendation**, the candidate it would do first. Then the skill stops and asks which candidate you want to explore. At that point you have decided nothing, and no code has changed.

## What happens after you pick one

When you pick a candidate, a [grilling](https://github.com/Yis-company/design-skills/blob/main/docs/productivity/grilling.md) session starts on it. It covers which side is the source of truth, which tier a token belongs in, whether a second use justifies a variant, and how the use sites migrate. The output is a decision, not a diff. You take it to [to-spec](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/to-spec.md) when engineers need a change specified, to [to-tickets](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/to-tickets.md) when it is a token migration or a set of fixes ready to slice, or record it as an ADR.

## Common questions

**What happened to the architecture review skill?**

This skill replaced it. The process is the same: scope by hot spots, a sub-agent walks the code, an HTML report, then grilling on the candidate you pick. What it looks for changed, from shallow code modules to drift from the design system.

**It grilled me for an hour about one idea instead of showing me options. Can I turn that off?**

Say so when you invoke it ("don't grill me, just show the report"). This was the most common complaint about the skill this one replaced, and the process has not changed. The design intent is that the report comes first, and the grilling starts only on a candidate you chose. Weaker [models](https://www.aihero.dev/ai-coding-dictionary/model) sometimes skip straight to an interview about the first idea they had. The skill does not yet have a documented no-grill mode.

**The code is right and the design file is out of date. Does it still call that drift?**

Yes, because drift is a gap, not a verdict on which side is wrong. The grilling step exists to settle that: the first question on any candidate is which side is the source of truth. If it is the code, the decision is to update the design source, and nothing goes to engineers.

**The report opened as unstyled raw HTML. What happened?**

The report loads Tailwind from a CDN, so it needs network access when you open it. If something blocks that script (an offline machine, a locked-down browser, a security hook demanding SRI hashes), the page breaks with no error. The agent cannot see it, because it never renders the page. As a workaround, ask for inline CSS instead of the CDN.

**It gave me twelve candidates. Do I work through them in the same session or start a new one?**

Use one candidate per session. If you work through several in one conversation, the [context window](https://www.aihero.dev/ai-coding-dictionary/context-window) fills with the report, the grilling and the glossary edits all at once. The report is only a temp file, so carry the candidate forward, not the file. Pick one, grill it, and take the decision into `/to-spec` or `/to-tickets`. Note the rest so you can pick them up separately later.

**How should I prompt it?**

Prompt it with the next thing you are designing, or the component family that bothers you most. A run with no prompt looks for hot spots on its own. That is fine for routine upkeep, but a direction makes the report actionable.

**Will it ever tell me the system is fine?**

Rarely, so know that before you start. The skill exists to output findings, so it tends to produce candidates instead of concluding that nothing is wrong. Use the strength badges to correct for this. If every candidate in a report is `Speculative`, the skill found nothing.

## It's working if

- The candidates name your product's concepts and your design system's tokens and components, not invented names.
- The candidates are in screens and components that changed recently, not in parts of the product nobody touches.
- Each card shows the drift as real swatches, type or components you can see, not only as prose.
- No code changed during the run. The only new file is the HTML report in your temp directory.
- It stops after the report and asks which candidate you want. It does not continue on its own.
- It does not propose a new variant or token on the strength of a single use.
- When you reject a candidate for a lasting reason, it offers to record an ADR, so the next run does not suggest it again.

## Where it fits

`audit-design-system` is **periodic maintenance**. You run it every few weeks, outside any chain, to queue up work, not to do it. Its neighbours:

- [design-system](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/design-system.md) owns the token, component and drift vocabulary every candidate uses.
- [grilling](https://github.com/Yis-company/design-skills/blob/main/docs/productivity/grilling.md) walks the decision tree after you choose a candidate.
- [domain-modeling](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/domain-modeling.md) keeps `GLOSSARY.md` and the ADRs current as you make the decision.
- [design-qa](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/design-qa.md) is the per-diff counterpart: it catches new drift as it is built, and this skill catches what got through.

Its output is a decision, which goes into the main flow at [to-spec](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/to-spec.md) or [to-tickets](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/to-tickets.md). For which skill fits a situation, [ask-design](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/ask-design.md) is the router over the whole set.
