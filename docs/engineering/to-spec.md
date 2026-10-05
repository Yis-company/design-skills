## What it does

`to-spec` turns the conversation you have just had, and the designs it was about, into a design **[spec](https://www.aihero.dev/ai-coding-dictionary/spec)**: flows, screen states, components and tokens, content, and accessibility. It publishes the spec to your issue tracker as a single issue that engineers and their agents build from.

It does not interview you. When you reach for it, the deciding is already done. So it synthesises what is known (from the thread, the design source, the codebase, your `GLOSSARY.md` and ADRs) and does not start a new round of questions. The spec records decisions you already made. It is not a place to make new ones.

## When to reach for it

You invoke this by typing `/to-spec`; the [agent](https://www.aihero.dev/ai-coding-dictionary/agent) won't reach for it on its own.

Reach for it when the design is settled and the build is too big to hand over as one ticket:

| Where you are | What to run |
| --- | --- |
| You haven't decided anything yet | [grill-with-docs](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/grill-with-docs.md) first |
| You want to react to working variants before deciding | [prototype](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/prototype.md) first |
| Decided, and it is one small screen change | Skip the spec: [to-tickets](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/to-tickets.md) reads the thread directly |
| Decided, and the work spans several flows or screens | `/to-spec`, then [to-tickets](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/to-tickets.md) |
| A [wayfinder](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/wayfinder.md) map has cleared | `/to-spec #<map_issue>` |

## Prerequisites

`to-spec` publishes the spec as an issue, so [setup-design-skills](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/setup-design-skills.md) must first configure a tracker and the triage-label vocabulary for this repo. Either kind of tracker works: a real tracker like GitHub, or local markdown files under `.scratch/`, which work with no extra setup.

You also need the design source: a link to the designs, plus exported frames or screenshots of the screens in scope. If setup recorded where designs live and how to export them, the agent uses that.

## The spec is a decision record

The spec exists because context windows end. You settled many things while [grilling](https://www.aihero.dev/ai-coding-dictionary/grilling): the flows, the choices you argued through, and what you deliberately refused. All of that is in one conversation that you are about to clear. The spec keeps it.

So the spec does not validate or decide anything. It records what you decided, in your project's own vocabulary, so an engineer's fresh session can pick up the work without you explaining it again. If the spec states something you never said, that is a defect.

## Inventory before prose

Before it writes anything, `to-spec` reads the code and builds the **component inventory**: the existing components, variants and tokens the feature uses, named as they are in code, plus the new components, variants and tokens the design needs. It checks that list with you. Anything in the design that matches nothing in code is either a new system piece or a one-off, and you decide which.

Other steps lean on that list. [to-tickets](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/to-tickets.md) turns every new token, component and variant into a ticket that blocks the screens using it. [design-qa](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/design-qa.md) checks the build against it, so a one-off nobody agreed to shows up as a finding. That is why the inventory conversation is worth taking seriously here, not after the build.

The same goes for **screen states**. For every screen the spec lists default, empty, loading, error, partial and overflow, each marked *designed* or *engineer's call*. A state nobody designed is still a decision; the spec makes it a visible one.

## Common questions

**Does it read my design file directly?**
No. The design source is a link plus exports, and the skill never talks to a design tool. If the agent can't open the link, it asks you for exported frames or screenshots. Export the states as well as the happy path, or the spec will mark them *engineer's call*.

**Why does the spec get the `ready-for-agent` label? I don't want an agent building off it.**
The label means "no further triage needed": the document is complete enough for an agent to work from. It marks an input, not a work order. But [AFK](https://www.aihero.dev/ai-coding-dictionary/afk) agents that poll for `ready-for-agent` cannot see that difference. They will try to build the whole spec in one run instead of picking up the ticket slices. This is the most-reported problem with the skill. Until it changes, ask engineers to exclude the parent spec in their AFK agent's prompt, or remove the label after `/to-tickets` has run.

**Why not go straight from grilling to `/to-tickets` and skip the spec?**
Often you should. The spec is worth its step only on multi-session work. Its value is that the tickets are disposable and the spec is not. Each ticket is sized for one fresh context window and then gets closed, while the spec stays as the one place that records the reasoning behind them. On a small change, that gives you nothing, and you pay for an extra synthesis step where the [model](https://www.aihero.dev/ai-coding-dictionary/model) can drift.

**Where did `/to-prd` go?**
It is this skill, renamed upstream in v1.1. "Spec" is now the one term used throughout, and the old `to-prd` slug no longer works, so reinstall under the new name. The spec is the destination and the decisions that fix it. The [tickets](https://www.aihero.dev/ai-coding-dictionary/ticket) are the steps that get there. If you change direction, delete the unfinished tickets and keep the spec.

**I just finished a wayfinder map. What do I feed it?**
Give it the main map issue, `/to-spec #<map_issue>`, not the individual decision tickets. [wayfinder](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/wayfinder.md) produces decisions spread across a map, not deliverables. `to-spec` collapses them into one document engineers can build from.

**Is the spec for me to review, or is it just for the agent?**
Mostly for the agent, and it reads that way: complete, dense, and full of references. Read the component inventory, the screen states and the out-of-scope section. In those places, a wrong decision is cheapest to catch now and most expensive to find in the build. There is no summary mode. If the spec surprises you, the grilling was too shallow; the spec is not too long.

**Do I keep the spec frozen once tickets start, or let the agent rewrite it?**
Nothing keeps it in sync. In practice it is a snapshot of what you knew at that moment, and it goes out of date the first time the build teaches you something. Treat it as disposable after the work ships. Your `GLOSSARY.md`, ADRs and the design system itself are the things meant to last.

**Will it check the tracker for related work, or cite the ADRs it's respecting?**
No to both. It reads and follows the ADRs for the area it touches, but it does not link them. It also does not search the tracker for overlapping issues before it writes, so a spec can duplicate work that someone already filed, and nothing warns you. If the area is busy, search the tracker yourself first.

**`/to-tickets` couldn't read my spec: it kept truncating.**
A tracker issue may not return a very large spec in full, and there is no local copy to use instead. To fix this, do not [clear](https://www.aihero.dev/ai-coding-dictionary/clearing) or [compact](https://www.aihero.dev/ai-coding-dictionary/compaction) between `/to-spec` and `/to-tickets`. Run them in the same window, and `/to-tickets` never has to fetch the spec again.

## It's working if

- It starts writing instead of asking you a new round of questions.
- It shows you the component inventory before it writes, using the names you'd find in code.
- Every flow step and every *designed* state links to a frame or export.
- You remember making every decision in it. It invented nothing to fill a section.
- The out-of-scope section lists real things. The things you refused are usually the most useful lines on the page.

## Where it fits

`to-spec` is a step in the main designer chain, on the branch where the work spans several sessions:

```txt
grill-with-docs → to-spec → to-tickets → engineers build → design-qa
```

Upstream, [grill-with-docs](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/grill-with-docs.md) makes the decisions that this skill only records, and a finished [wayfinder](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/wayfinder.md) map joins the chain here. Downstream, [to-tickets](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/to-tickets.md) cuts the spec into flow and screen tickets, and [design-qa](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/design-qa.md) checks the finished build against it. When you're unsure which skill or flow fits, [ask-design](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/ask-design.md) routes you.
