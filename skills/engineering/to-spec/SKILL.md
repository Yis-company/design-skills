---
name: to-spec
description: "Turn the current conversation and its designs into a design spec (flows, screen states, components, tokens, accessibility) and publish it to the project issue tracker: no interview, just synthesis of what you've already discussed."
disable-model-invocation: true
---

This skill takes the current conversation, the design source and your understanding of the codebase, and produces a design spec that engineers (and their agents) build from. Do NOT interview the user; just synthesize what you already know.

The issue tracker and triage label vocabulary should have been provided to you. If not, tell the user to run `/setup-design-skills`.

## Process

1. Gather the design source: the link plus exported frames or screenshots for every screen in scope. Work from links and exports only; don't call a design tool's API or MCP. If you can't open a link, ask the user for exports. The project may record where designs live and how to export them; use that if it exists.

2. Explore the repo to understand the current state of the UI, if you haven't already. Read code, never change it. Use the project's domain glossary vocabulary throughout the spec, and respect any ADRs in the area you're touching.

3. Confirm the component inventory. Call the Skill tool with "design-system" for the vocabulary. Then list, by their code names:

   - existing components, variants and tokens the feature will use (semantic tokens over primitives)
   - new components, new variants of existing components, and new tokens the design needs

   Check the list with the user. Anything in the design that matches nothing in code is either a new system piece or a one-off: ask which.

4. Write the spec using the template below, then publish it to the project issue tracker. Apply the `ready-for-agent` triage label - no need for additional triage.

<spec-template>

## Problem Statement

The problem that the user is facing, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

## User Stories

A LONG, numbered list of user stories. Each user story should be in the format of:

1. As an <actor>, I want a <feature>, so that <benefit>

<user-story-example>
1. As a mobile bank customer, I want to see balance on my accounts, so that I can make better informed decisions about my spending
</user-story-example>

This list of user stories should be extremely extensive and cover all aspects of the feature.

## Flows

Each flow as numbered steps, every step linked to its frame or export in the design source.

## Screen states

For each screen: default, empty, loading, error, partial and overflow. Mark each state *designed* (link the frame) or *engineer's call* (one line on what is expected).

## Components and tokens

The confirmed inventory:

- Existing components, variants and tokens, named as in code
- New components, with their variants and slots
- New variants of existing components
- New tokens, with their tier (primitive, semantic, component)

## Content

Which copy is final and which is placeholder. Placeholder copy is listed explicitly so it doesn't ship.

## Responsive, interaction and accessibility

Breakpoints and what changes at each, motion, focus order, keyboard behaviour, contrast and target size.

## Design decisions

The choices that were made and why, including the alternatives you rejected.

Do NOT include file paths or code snippets anywhere in the spec. They may end up being outdated very quickly. Name components and tokens as they appear in code instead.

Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, variant props, token values), inline it within the relevant decision and note briefly that it came from a prototype. Trim to the decision-rich parts, not a working demo, just the important bits.

## How design QA will check it

Which screens and states `design-qa` compares against which frames, at which breakpoints (the Fidelity axis), and which components and tokens it expects the build to use (the System axis).

## Out of Scope

A description of the things that are out of scope for this spec.

## Further Notes

Any further notes about the feature.

</spec-template>
