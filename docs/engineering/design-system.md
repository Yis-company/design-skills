## What it does

`design-system` fixes the words you use to talk about a design system: **token** (in primitive, semantic and component tiers), **component**, **variant**, **slot**, **primitive vs composite**, **drift**, **one-off**, **source of truth**. It defines each one precisely, names the loose substitutes to avoid ("style", "widget", "modifier", "inconsistency"), and states the handful of principles that follow from them. The same words cover the design source and the code, so a designer and an engineer mean the same thing by "variant."

It is a reference, not a process. It runs no loop, produces no artifact, and never stops to ask you a question. [design-qa](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/design-qa.md), [audit-design-system](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/audit-design-system.md) and [to-spec](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/to-spec.md) all use its vocabulary. On its own, it gives you the words and stops.

## When to reach for it

Type `/design-system`, or the agent reaches for it automatically when a task needs the vocabulary.

Reach for it when you are naming or judging a piece of the system: whether a value should be a token and in which tier, whether a difference earns a variant, whether two components are really one. Also use it to settle an argument about what a design-system word means.

Several skills are close to it. Pick by the problem you have:

| The problem | The skill |
|---|---|
| The words of the design system: token tiers, variant, slot, drift | `design-system` |
| The *words of the product*: "account" means three things, two people mean different things by "cancellation" | [domain-modeling](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/domain-modeling.md) |
| You don't yet know *where* the system has drifted | [audit-design-system](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/audit-design-system.md) (the survey that finds candidates) |
| You want a decision argued with, not just named | [grilling](https://github.com/Yis-company/design-skills/blob/main/docs/productivity/grilling.md) |

## The vocabulary

The glossary is the skill. It defines every term against the others, and gives each one the word it replaces.

| Term | What it means | Don't say |
|---|---|---|
| **Token** | A named design decision (colour, space step, type style, radius, duration) stored as data so the design source and the code read the same value. | variable, style, constant |
| **Primitive / semantic / component token** | The three tiers. A primitive is a raw value on a scale (`blue-600`). A semantic token is a role pointing at a primitive (`color-text-danger`). A component token is a semantic decision scoped to one component (`button-primary-bg`). | none |
| **Component** | A reusable piece of UI with a name, an API and variants, present in both the design source and the code library. | widget, element, module |
| **Variant** | A named, supported configuration of a component, part of its API. A style override passed in from outside is not one. | modifier, flavour |
| **Slot** | A named place inside a component where the caller supplies content. | children |
| **Primitive vs composite** | A primitive component contains no other system components; a composite is assembled from them. | none |
| **Drift** | Any gap between the design system and what ships, measured against the source of truth. | inconsistency, tech debt |
| **One-off** | A value, variant or component used in exactly one place outside the system. | custom, special case |
| **Source of truth** | The one place a given decision is defined; everything else is a copy that can drift. | master |

## The four principles

- **Semantic tokens at use sites.** A screen reads `color-text-danger`, never `red-600` and never a hex. A use site that reaches for a primitive has skipped the decision the semantic tier records.
- **The deletion test.** Imagine deleting a component and inlining it everywhere. If the screens barely change, it was a pass-through. If the same decisions reappear, slightly different, at every site, it was earning its keep.
- **One use is a hypothetical variant. Two uses is a real one.** Don't add a variant, token or component for a single screen. Leave the one-off visible until a second use shows up.
- **The component API is the design contract.** The props, variants and slots in code are what designers and engineers agree on. When the design-source component and the code component disagree, that is drift, and one side changes to match the source of truth.

## Common questions

**What happened to the deep-module vocabulary skill?**

This skill replaced it. The shape is the same (a glossary, a few principles, the substitutes to avoid), but the subject moved from code modules to the design system. The deletion test and the "one use is hypothetical, two is real" rule carried over, applied to components and variants instead of modules and seams.

**I pointed a session at it and it started auditing things I never asked about.**

That is the known failure of a reference skill: it has no process and no stopping rule, so an agent told to "go" invents one. This skill has none of the guardrails a driver skill has (checkpoints, one question at a time, no auto-advance). Invoke a driver instead (`/audit-design-system`, `/to-spec` or `/grill-with-docs`) and let it use this vocabulary.

**Our team says "styles" or "variables", not "tokens". Do we have to switch?**

In conversation with the agent, use the skill's terms, so every skill reads the same words the same way. Your design tool can keep its own labels. If your product needs extra terms of its own, they belong in `GLOSSARY.md` through [domain-modeling](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/domain-modeling.md), not in this skill: the glossary here is small on purpose, because a term nobody uses consistently is worse than no term.

## It's working if

- The conversation stops producing "style", "widget" and "inconsistency", and starts producing "semantic token", "component" and "drift".
- Someone can point at a proposed variant and say whether it has its second use, without hedging.
- A hardcoded value in review gets named with the semantic token it should have been, not just flagged.
- A disagreement between the design file and the code ends with a named source of truth.
- Invoking it does not start a session. If the agent begins reading files and proposing fixes from `/design-system` alone, it has mistaken the reference for a driver.

## Where it fits

`design-system` is a **reach-for-it-anytime standalone**, and the vocabulary layer underneath the design skills rather than a step in any chain. Its closest neighbour is [domain-modeling](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/domain-modeling.md), the parallel reference for the *product*'s words rather than the system's. [audit-design-system](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/audit-design-system.md) and [design-qa](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/design-qa.md) write every finding in this glossary, so they find the drift and you discuss it in this skill's words. When you're unsure which skill or flow fits, [ask-design](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/ask-design.md) routes you.
