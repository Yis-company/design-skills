---
name: ask-design
description: Ask which skill or flow fits your situation. A router over the skills in this repo.
disable-model-invocation: true
---

# Ask Design

You don't remember every skill, so ask.

A **flow** is a path through the skills. Most paths run along one **main flow**, and two **on-ramps** merge onto it. Everything else is standalone, or a vocabulary layer that runs underneath.

## The main flow: idea → design QA

The route most work travels. You have a design idea and want it built to match.

1. **`/grill-with-docs`** sharpens the idea by interview. Start here whenever you are **working in a working directory**: it's stateful, retaining what it learns in `GLOSSARY.md` and ADRs, design-system terms included. (No working directory? Use `/grill-me` instead, covered under Standalone. Both run the same `/grilling` primitive; `grill-with-docs` is the one that leaves a paper trail, which makes it the better of the two whenever a repo is there to leave it in.)
2. **Branch: can you settle every question in conversation?** If a question needs something you can see and click (a state model, a flow, a UI you have to try), detour through a prototype, bridged by **`/handoff`** in both directions (a prototype lives in its own directory, which is exactly what `/handoff` is for; see Phase boundaries):
   - **`/handoff`** out, then open a fresh session against that file,
   - **`/prototype`** to answer the question with throwaway code you run locally,
   - **`/handoff`** back what you learned, and reference it from the original idea thread.
3. **`/to-spec`** turns the thread into a **design spec**: flows, screen states, components and tokens, content, accessibility, linked to the frames in your design source. Then **`/to-tickets`** splits it into tickets engineers can build, one flow or screen per ticket, each declaring its **blocking edges** (new tokens and components block the screens that use them). For a change small enough for one ticket, skip the spec and run `/to-tickets` on the conversation.
4. **Engineers build.** That happens outside this set: you hand over the tickets and stay read-only on production code.
5. **`design-qa`** checks the build on two axes: **Fidelity** (does it match the spec and the design source, every state included?) and **System** (does it use the design system's tokens and components?). It's model-invoked, so the agent reaches for it whenever you ask it to check a branch or PR against the design. Its findings name the file and the expected vs actual, so engineers can act on them without a meeting.

   When a change goes up as a pull request, **`/pr`** shapes the body: the smallest visual that shows the change, before/after evidence, and a one-way or two-way door call. It's model-invoked, so the agent reaches for it whenever it writes a PR.

6. **`/retro`** closes the loop. After a round that went sideways, it looks back over the session and suggests changes to the agent's **environment**, not the design: navigation pointers, automated checks, the standards `design-qa` enforces, steering files. The next round then starts from a better environment.

### Context hygiene

Keep steps 1–3 in **one unbroken context window** (don't compact or clear until after `/to-tickets`) so the grilling, spec, and tickets all build on the same thinking. `design-qa` starts fresh, working from the spec and the diff. Run `/retro` in the session it's looking back on, before you clear; after clearing, point it at that session's log instead.

The limit on this is the **[smart zone](https://www.aihero.dev/ai-coding-dictionary/smart-zone)**: the window (~150k tokens on state-of-the-art models) within which the model still reasons sharply. If a session approaches it before `/to-tickets`, don't push on degraded; `/compact` at the nearest phase boundary and carry on (see Phase boundaries).

## On-ramps

A starting situation that generates work, then merges onto the main flow.

- **UI bugs and requests piling up** → **`/triage`**. It moves issues through triage roles and produces agent-ready issues for engineers to pick up.

  Triage is only for issues **you didn't create**: bug reports, incoming feature requests, anything that arrives raw. Tickets that `/to-tickets` produced are already agent-ready, so **don't triage them**.

- **Something's broken and the cause is hidden in code** → **`/diagnosing-bugs`**. For the hard ones: the layout that breaks only sometimes, the regression that crept in between two known-good states. It refuses to theorise until it has a **tight feedback loop**, then fixes with a regression test, so it's for sessions where a code fix is in scope. Read-only? Use it to pin down the cause, then hand the finding to an engineer through `/triage`.

- **A huge, foggy effort: a new product area or a redesign too big for one session** → **`/wayfinder`**, the most cognitively demanding flow here. When the way from here to the destination isn't visible yet, it charts a **shared map** of **decision tickets** on the issue tracker and resolves them one at a time, producing **decisions, not deliverables**, until the fog is pushed back and the way is clear. Where **`/grill-with-docs`** sharpens an idea you can hold in one session, wayfinder is for the idea you can't, and it's slower and denser, so save it for exactly that, never a well-scoped feature.

  When the map clears, **it hands off, it doesn't build**: merge onto the main flow at **`/to-spec`**, which collapses the map's linked decisions into a design spec, then `/to-tickets` as usual.

## Design-system health

Not feature work, just upkeep.

- **`/audit-design-system`** runs whenever you have a spare moment. It surfaces **drift** between the code and the design system: hardcoded values where a token exists, near-duplicate components, one-off variants. Picking a candidate ends in a decision you take to `/to-spec` or `/to-tickets` for engineers, or record as an ADR. It's the survey that finds the candidates; **`design-system`** (below) is the vocabulary you discuss them in.

## Vocabulary underneath

Two model-invoked references that run *beneath* the other skills, each the single source of truth for its vocabulary. Reach for them directly when the **words**, not the process, are the problem; or let the skills above pull them in.

- **`/domain-modeling`**: sharpen the project's *domain* language: challenge a fuzzy term, resolve an overloaded word ("account" doing three jobs), record a hard-to-reverse decision as an ADR. It's the active discipline `/grill-with-docs` drives to keep `GLOSSARY.md` a clean glossary.
- **`design-system`** is the design-system vocabulary (token, component, variant, slot, drift, one-off) for talking about the system's *shape*: semantic tokens at use sites, a variant earning its place by a second use, the component API as the design contract. `to-spec`, `design-qa` and `/audit-design-system` all speak it.

## Phase boundaries

A **phase** is a chunk of work inside a session: the grilling, the implementation, the QA. At the **boundary** between two of them you have five options, and picking between them is the fuzziest decision in this whole map:

- **Continue**: stay put. Costs nothing, loses nothing.
- **`/clear`**: empty the window, when nothing here matters to what's next.
- **`/handoff`** writes a portable markdown file. Narrow: only for a **new harness**, a **new directory**, a **colleague**, or forking a side task **mid-phase**. What it buys is portability.
- **Subagent**: send a tightly-scoped task to its own window and get a report back.
- **`/compact`** compresses this context and seeds a fresh session with it. The **default**, at the bottom of the tree rather than the first reach.

Read [PHASE-BOUNDARIES.md](PHASE-BOUNDARIES.md) for the ordered tree: the five questions, the reasoning behind each branch, and why the primary-source cost makes **Continue** the one to rule out first. Make the decision **at** a boundary; mid-phase, continue or split the rest into subagents.

## Standalone

Off the main flow entirely.

- **`/grill-me`**: the same relentless interview as `/grill-with-docs`, but **stateless**: it saves nothing locally and builds no `GLOSSARY.md`. Reach for it when you are **not working in a working directory** (sharpening a plan, a design, a piece of writing, anything with no repo under it). If you are in a working directory, use `/grill-with-docs` instead: it runs the same interview and leaves a paper trail, so it is strictly the better one.
- **`/grilling`** is the interview primitive itself: rounds, the frontier, facts are the agent's job and decisions are yours. `/grill-me` and `/grill-with-docs` are the two named ways in, and `/triage`, `/wayfinder` and `/audit-design-system` all run it internally. Reach for it directly only when you want the interview with no wrapper around it.
- **`/prototype`** is a small, throwaway program you run locally that answers one design question: does this state model feel right, or what should this UI look like. Throwaway is a constraint on how the code is written, not a promise to destroy it: the answer folds into the spec, and the prototype itself is kept as a **primary source** on a `prototype/<name>` branch out of main, pointed at from the spec. It's the detour in step 2 of the main flow, but reach for it any time a design question is hard to settle on paper.
- **`/research`**: delegate reading legwork to a **background agent**: it investigates a question against **primary sources**, then leaves a cited Markdown file in the repo. Keep working while it reads. Pattern research and accessibility guidance are good fits. The file it produces is something to take *into* the main flow at `/grill-with-docs`, since research feeds the thinking rather than replacing it.
- **`/to-questionnaire`** comes in when the thing blocking you isn't in your head or the codebase but in **someone else's**, and it writes them a questionnaire to fill in. It's the inverse of `/grill-me`: instead of interviewing you about the subject, it interviews you about the **send** (who it's going to, what you need back) and aims the questions at the gap. What comes back is material for `/grill-with-docs` or `/to-spec`.
- **`/wizard`** is for the steps only a **human** can take: provisioning infrastructure, setting up credentials or CI secrets, clicking through an unfamiliar third-party dashboard, running a one-off migration or cutover. It generates an interactive bash script that opens each URL, captures each value, and writes it into `.env` and GitHub secrets, so the procedure stops being something you re-explain to an agent every time. Model-invoked, so the agent reaches for it the moment it hits a wall only you can pass. If the agent could just do it itself, it should; this is for where a human is genuinely in the loop.
- **`/wait-what`** is the corrective for a message that didn't land. Use it mid-conversation, inside any other skill, and the agent re-pitches what it just said with the context you were missing, in plain English, using the `GLOSSARY.md` vocabulary. It works after the fact; `/grill-with-docs` is the upfront cure, because a shared language agreed early is what stops the jargon arriving at all.
- **`/teach`**: learn a concept over multiple sessions, using the current directory as a stateful workspace.
- **`/writing-for-agents`** is the reference for writing documents agents consume: skills, AGENTS.md, pointed-at docs.

## Precondition

**`/setup-design-skills`**: run before your first flow to configure the issue tracker, triage labels, doc layout, and design source the other skills assume. Custom issue trackers also work.
