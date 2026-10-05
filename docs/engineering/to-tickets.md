## What it does

`to-tickets` takes a design [spec](https://www.aihero.dev/ai-coding-dictionary/spec), a plan, or the conversation you are in, and breaks it into a set of **[tickets](https://www.aihero.dev/ai-coding-dictionary/ticket)** on your issue tracker. Each ticket declares its **blocking edges**: the other tickets that have to finish before it can start.

Every ticket is a **vertical slice**: one flow or one screen, carried through all of its screen states, that someone can open and walk through as soon as it lands. This is what makes the skill different from the obvious way to split design work, which is to cut by state or by layer ("all the empty states", "all the styling") and integrate at the end. It also sizes each ticket to fit in a single fresh [context window](https://www.aihero.dev/ai-coding-dictionary/context-window), because a [session](https://www.aihero.dev/ai-coding-dictionary/session) that has never seen your spec will pick the ticket up.

## When to reach for it

You invoke this by typing `/to-tickets`. The [agent](https://www.aihero.dev/ai-coding-dictionary/agent) won't reach for it on its own.

| Where you are | What to run |
| --- | --- |
| You have a spec issue and the build spans several sessions | `/to-tickets`, or `/to-tickets #<spec_issue>` |
| The design is only in the conversation, never written up | `/to-tickets` reads the thread directly, no spec needed |
| Nothing is decided yet | [grill-with-docs](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/grill-with-docs.md), then [to-spec](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/to-spec.md) |
| A [wayfinder](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/wayfinder.md) map has cleared | [to-spec](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/to-spec.md) first, to collapse the map, then `/to-tickets` |

Tickets that `to-tickets` produced are agent-ready by construction. Don't run [triage](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/triage.md) over them. Triage is for work that arrived from someone else.

## Prerequisites

`to-tickets` publishes into a tracker, so [setup-design-skills](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/setup-design-skills.md) must have configured one for this repo, along with the triage-label vocabulary. Either kind works: a real tracker like GitHub or Linear, or local markdown files under `.scratch/`, which work with no extra setup.

Every acceptance criterion points at a frame or export, so have the design source to hand: the link, plus exports if the agent can't open it.

## Flows and screens, not states or layers

A **horizontal** slice ships one cut across the work: every empty state, or every token change. Nothing can be demoed until every slice has landed, and each ticket's acceptance criteria reach into screens another ticket owns. A **vertical** slice ships one flow or screen through all its states at once, so it is checkable alone and owns everything it grades.

The exception is the design system itself. **System first**: each new token, component or variant is its own ticket, and it blocks every screen that uses it. Screens then build on real system pieces instead of growing one-offs that someone has to clean up later.

Before anything is published, `to-tickets` presents the breakdown as a numbered list and quizzes you on it: is the granularity right, are the blocking edges real, does the system work come first, should anything merge or split. Nothing reaches the tracker until you approve, and that quiz is the place to push back.

## Blocking edges

The edges are the point of the artifact. They work in two ways, depending on the tracker:

| Tracker | Where the edges live | How you work them |
| --- | --- | --- |
| Local markdown | Text in one file per ticket under `.scratch/<feature>/issues/<NN>-<slug>.md`, numbered blockers-first | Top to bottom, by hand |
| A real tracker (GitHub, Linear) | Native blocking links, or sub-issues where the tracker has them | Any ticket whose blockers are done is on the **frontier** and can be grabbed |

The edges live in the ticket either way. The tracker only decides whether anything can act on them in parallel. `to-tickets` produces the artifact; running it is the engineers' job, not the skill's.

## Token migrations

One shape breaks the vertical-slice rule. Replacing a token used across the codebase (renaming it, retiring a primitive, splitting a semantic token in two) touches every screen at once, so no single slice can carry it.

`to-tickets` sequences that as **expand–contract** instead:

- **Expand**: add the new token beside the old, so nothing changes.
- **Migrate**: move usages over in batches (per screen area or per component), one ticket per batch, each blocked by the expand. Each batch names the screens it touches and how they should look afterwards.
- **Contract**: remove the old token once nothing uses it, in a ticket blocked by every migrate batch.

## Common questions

**It produced twelve tickets for a three-line change.**
Over-decomposition is the most reported problem with this skill, and many users see it. The [model](https://www.aihero.dev/ai-coding-dictionary/model) defaults to atomic units and loses the grouping that would make them meaningful. The quiz step is where you fix this. Ask it to merge tickets, and it will. There is also a lower limit. If the whole change fits in one context window, one ticket is enough.

**The tickets came out one per state or one per layer: all the empty states in one, all the styling in another.**
This is the failure the vertical-slice rule is written against, and the skill still produces it sometimes. Catch it at the quiz step by asking one question per ticket: what can I walk through when this is done? A ticket with no answer is a horizontal slice. New tokens, components and variants are the one exception, and they belong at the front of the order.

**Why are the new tokens and components separate tickets?**
Because screens that land before their system pieces exist end up hardcoding values and copying components, and [design-qa](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/design-qa.md) then flags every one of them as drift. Putting the system work first, as blockers, means the screens can use the real thing from the start.

**On GitHub the tickets weren't created as sub-issues of the spec issue.**
This is a known bug, and it is not fixed. It has been reported across a dozen runs and several models, [most fully in issue #554](https://github.com/mattpocock/skills/issues/554), and it is worse on Codex than on Claude. `gh` has supported this natively since v2.94: `gh issue create --parent <n>`, and `gh issue edit <parent> --add-sub-issue <n>` after the fact. Until the tracker template prefers those, the reliable fix is to add the parent links yourself after a run.

**"Blocked by" was written into the issue body instead of a real blocking link.**
This is the same kind of problem, [reported in issue #513](https://github.com/mattpocock/skills/issues/513), where the agent even stated that GitHub has no native blocking relationship at all. It does: `gh issue create --blocked-by 12,15`. Because the skill publishes blockers first, their numbers are always available at creation time. The body text is meant to be the fallback for trackers with no native edge, not the default.

**Where do the local tickets go? The v1.1 notes said a root-level `tickets.md`.**
They did, and that was a bug. A single shared file also caused race conditions when parallel agents wrote to it. Local mode now writes one file per ticket under `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, in dependency order. The `NN` prefix is a real ticket ID, so you can refer to ticket `03` instead of retyping a long title.

**It kept truncating when it tried to read my spec.**
A very large spec can outgrow what a tracker issue serves back cleanly. There is no local copy to fall back on, so the agent spends [tool calls](https://www.aihero.dev/ai-coding-dictionary/tool-call) fetching chunks again and never reaches the end. Don't [clear](https://www.aihero.dev/ai-coding-dictionary/clearing) or [compact](https://www.aihero.dev/ai-coding-dictionary/compaction) between `/to-spec` and `/to-tickets`. Run them in the same context window and the agent never has to fetch the spec back.

**The acceptance criteria graded nothing: some passed before any work was done.**
The template asks for criteria and says nothing about whether they can fail, so this happens. Three shapes recur: a criterion already true in the current build, a criterion that only work in another ticket can satisfy, and one that restates the request rather than pointing at a frame. Vertical slicing prevents most of it, because a slice that delivers a new screen or state fails on the current build by construction. The check is still worth doing by hand: for each criterion, name what you would see on screen that shows it false.

**The tickets are published. How do I actually run them?**
The skill stops at the artifact, and there is no auto-dispatch mode. Engineers (or their agents) pick up tickets on the frontier, one fresh context per ticket. Nothing closes a ticket automatically when the work lands, so whoever finishes it updates its state. When a ticket lands, [design-qa](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/design-qa.md) checks it against the frames its criteria point at.

## It's working if

- Every ticket has an answer to "what can I walk through when this is done?", and the answer is a flow or a screen, not a state or a layer.
- Every acceptance criterion points at a frame or export.
- New tokens, components and variants sit at the front of the order, blocking the screens that use them.
- The list comes back to you numbered, with a "Blocked by" line on each, before anything is published.
- Nothing in a ticket body is a file path or a line number, except a snippet a prototype produced.
- Each ticket reads like something a fresh session could finish without you in the room.

## Where it fits

`to-tickets` is a step in the main designer chain:

```txt
grill-with-docs → to-spec → to-tickets → engineers build → design-qa
```

Upstream is [to-spec](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/to-spec.md), which hands it a settled spec and component inventory to slice against. Keep both in one context window, with no clear between them. Downstream, engineers build one ticket per fresh session outside this skill set, and [design-qa](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/design-qa.md) checks each finished slice against its frames and the design system. When you're unsure which skill or flow fits, [ask-design](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/ask-design.md) routes you.
