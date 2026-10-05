---
name: design-system
description: Shared vocabulary for design systems in code. Use when the user wants to name or judge tokens, components, variants or slots, decide whether a one-off deserves a variant, spot drift between the design source and the code, or when another skill needs the design-system vocabulary.
---

# Design System

Talk about a design system in one set of words, whether the thing in question lives in the design source or in the code. Use this language and these principles wherever a token, component or variant is being named, judged or proposed. The aim is consistency for users and fewer one-offs for the people who keep the system.

## Glossary

Use these terms exactly. Consistent language is the whole point.

**Design system**: the tokens and components a product is built from, plus the rules for using them. It lives twice: once in the design source, once in code. _Avoid_: UI kit, style guide (each names only part of it).

**Token**: a named design decision (a colour, a space step, a type style, a radius, a duration) stored as data so both the design source and the code read the same value. Tokens come in three tiers:

- **Primitive token**: a raw value on a scale, named for what it is (`blue-600`, `space-4`).
- **Semantic token**: a role that points at a primitive, named for what it is for (`color-text-danger`, `space-inset-card`).
- **Component token**: a semantic decision scoped to one component (`button-primary-bg`).

_Avoid_: variable, style, constant (all fine words for the storage, wrong words for the decision).

**Component**: a reusable piece of UI with a name, an API and a set of variants, present in both the design source and the code library. _Avoid_: widget, element, module.

**Variant**: a named, supported configuration of a component (`size="sm"`, `tone="danger"`). A variant is part of the component's API. A style override passed in from outside is not a variant. _Avoid_: modifier, flavour.

**Slot**: a named place inside a component where the caller supplies content (an icon, a trailing action, a body). Slots let one component cover many layouts without a variant for each. _Avoid_: children (too narrow, there may be several slots).

**Primitive vs composite**: a **primitive** component has no other system components inside it (Button, Icon, Text). A **composite** component is assembled from primitives and other composites (Dialog, Card). Do not confuse a primitive component with a primitive token.

**Drift**: any gap between the design system and what ships: a hardcoded value where a token exists, a near-duplicate component, a component whose code API no longer matches its design-source counterpart. Drift is measured against the source of truth. _Avoid_: inconsistency, tech debt (too vague to act on).

**One-off**: a value, variant or component used in exactly one place outside the system. A one-off is either drift to fold back in or the first use of something the system lacks. _Avoid_: custom, special case.

**Source of truth**: the one place a given decision is defined, which every other place reads from. For each kind of decision (token values, component API, copy) the team names one; the rest are copies that can drift. _Avoid_: master, canonical (unless the team uses them for exactly this).

## Principles

- **Semantic tokens at use sites.** A screen or component reads `color-text-danger`, never `red-600` and never `#d92d20`. Primitives exist to feed semantic tokens. A use site that reaches for a primitive has skipped the decision the semantic tier records.
- **The deletion test.** Imagine deleting the component and inlining it at every use. If the screens barely change, it was a pass-through that adds API without consistency. If the same decisions reappear, slightly different, at N sites, it was earning its keep.
- **One use is a hypothetical variant. Two uses is a real one.** Do not add a variant, token or component for a single screen. Leave the one-off visible, and promote it when a second use shows up.
- **The component API is the design contract.** The props, variants and slots in code are what designers and engineers agree on. When the design-source component and the code component disagree on a variant or slot, that is drift, and one side has to change to match the source of truth.

## Relationships

- A **Semantic token** points at exactly one **Primitive token**; a **Component token** points at a **Semantic token**.
- A **Component** exposes **Variants** and **Slots**; together they form its API.
- A **Composite** contains **Primitives** and other **Composites**.
- **Drift** is measured against the **Source of truth**; a **One-off** is drift until a second use makes it a candidate for the system.

## Rejected framings

- **Tokens as a naming scheme for CSS variables**: tokens are decisions shared by the design source and the code. The storage format is a detail.
- **Every visual difference deserves a variant**: that turns the API into a style dump. A difference with one use stays a one-off until a second use appears.
- **Pixel-perfect as the definition of fidelity**: the design system is the contract. A build that uses the right semantic token at a different rendered value from a stale export is correct, and the export is the drift.
