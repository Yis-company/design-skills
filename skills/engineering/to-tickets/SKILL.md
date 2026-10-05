---
name: to-tickets
description: Break a design spec, plan, or the current conversation into tickets, one flow or screen through all its states per ticket with visual acceptance criteria, each declaring its blocking edges, published to the configured tracker (edges as text in one file per ticket locally, or native blocking links on a real tracker).
disable-model-invocation: true
---

# To Tickets

Break a spec, plan, or conversation into a set of **tickets**: vertical slices, each one flow or screen carried through all of its screen states, each declaring the tickets that **block** it.

The issue tracker and triage label vocabulary should have been provided to you. If not, tell the user to run `/setup-design-skills`.

## Process

### 1. Gather context

Work from whatever is already in the conversation context. If the user passes a reference (a spec path, an issue number or URL) as an argument, fetch it and read its full body and comments.

Collect the design source the spec points at: the links and the exported frames or screenshots. Work from links and exports only; don't call a design tool's API or MCP. If you can't open a link, ask the user for exports. The project may record where designs live and how to export them; use that if it exists.

### 2. Explore the codebase (optional)

If you have not already explored the codebase, read it (never change it) to see which components, variants and tokens already exist, by their code names. Ticket titles and descriptions should use the project's domain glossary vocabulary, and respect ADRs in the area you're touching.

Find the system work the screens depend on: new tokens, new components, new variants. That work comes first.

### 3. Draft vertical slices

<vertical-slice-rules>

- Each slice is one flow or one screen, carried through all of its screen states (default, empty, loading, error, partial, overflow) and every layer it needs to work: vertical, NOT a horizontal slice such as "all the empty states" or "all the styling"
- A completed slice is demoable on its own: someone can open the build and walk through it
- Each slice is sized to fit in a single fresh context window
- System first: each new token, component or variant is its own ticket, blocking the screens that use it

</vertical-slice-rules>

Give each ticket its **blocking edges**: the other tickets that must complete before it can start. A ticket with no blockers can start immediately.

Acceptance criteria are visual and behavioural: what someone sees or can do in the running build. Each one points at the frame or export it is checked against. Cover every screen state in the slice, and mark states the spec left as *engineer's call*.

**Token migrations are the exception to vertical slicing.** Replacing a token used across the codebase (rename it, retire a primitive, split a semantic token in two) touches every screen at once, so no single slice can carry it. Sequence it as **expand–contract**: add the new token beside the old, so nothing changes; migrate usages in batches (per screen area or per component), each batch its own ticket blocked by the add; then remove the old token in a ticket blocked by every batch. Each batch's criteria name the screens it touches and how they should look afterwards (unchanged, or matching the new frames).

### 4. Quiz the user

Present the proposed breakdown as a numbered list. For each ticket, show:

- **Title**: short descriptive name
- **Blocked by**: which other tickets (if any) must complete first
- **What it delivers**: the flow or screen, and the states someone can walk through when it lands

Ask the user:

- Does the granularity feel right? (too coarse / too fine)
- Are the blocking edges correct: does each ticket only depend on tickets that genuinely gate it, and does every new token, component or variant come before the screens that use it?
- Should any tickets be merged or split further?

Iterate until the user approves the breakdown.

### 5. Publish the tickets to the configured tracker

Publish the approved tickets. **How** depends on the tracker `/setup-design-skills` configured; the tickets are the same either way, only the shape of the blocking edges changes:

- **Local files** → write one file per ticket under `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, numbered from `01` in dependency order (blockers first). Each file's "Blocked by" lists the numbers/titles it depends on. Use the per-ticket file template below: one ticket per file, never a single combined file.
- **A real issue tracker (GitHub, Linear, …)** → publish one issue per ticket in dependency order (blockers first) so each ticket's blocking edges can reference real identifiers. Use the platform's native blocking / sub-issue relationship where it has one; otherwise set each ticket's "Blocked by" to the blocking issues. Apply the `ready-for-agent` triage label unless instructed otherwise; the tickets are agent-grabbable by construction.

Work the **frontier**: any ticket whose blockers are all done. For a purely linear chain that means top to bottom.

Do NOT close or modify any parent issue.

<local-ticket-template>

# <NN>: <Ticket title>

**What to build:** the flow or screen this ticket makes work, and its states, from the user's perspective, not a layer-by-layer build list.

**Blocked by:** the numbers/titles of the tickets that gate this one, or "None (can start immediately)".

**Status:** ready-for-agent

- [ ] Visual or behavioural criterion 1 (frame or export it is checked against)
- [ ] Visual or behavioural criterion 2 (frame or export it is checked against)

</local-ticket-template>

<issue-template>

## Parent

A reference to the parent issue on the tracker (if the source was an existing issue, otherwise omit this section).

## What to build

The flow or screen this ticket makes work, and its states, from the user's perspective, not a layer-by-layer build list.

## Acceptance criteria

- [ ] Visual or behavioural criterion 1 (frame or export it is checked against)
- [ ] Visual or behavioural criterion 2 (frame or export it is checked against)

## Blocked by

- A reference to each blocking ticket, or "None (can start immediately)".

</issue-template>

In either form, avoid specific file paths or code snippets: they go stale fast. Name components and tokens as they appear in code instead. Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, variant props, token values), inline it and note briefly that it came from a prototype. Trim to the decision-rich parts, not a working demo, just the important bits.
