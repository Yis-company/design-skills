# Design Skills

[![skills.sh](https://skills.sh/b/Yis-company/design-skills)](https://skills.sh/Yis-company/design-skills)

Agent skills for designers who can read the codebase but do most of their work in Figma (or Penpot, Sketch, or anything similar).

The skills are read-only on production code. They turn design decisions into specs and tickets engineers can build from, check the built UI against the design and the design system, and find where the code has drifted from the system. You can still run the app and build throwaway prototypes locally.

Forked from [Matt Pocock's skills](https://github.com/mattpocock/skills) (MIT). They stay small, easy to adapt, and composable, and they work with any model.

## Installation

Two ways in. **The [Claude Code plugin](https://code.claude.com/docs/en/plugins)** installs the whole set as a managed, read-only bundle. **[skills.sh](https://skills.sh/Yis-company/design-skills)** copies editable skill files into your project, so you can make them your own. Pick one: installing both leaves you with every skill twice.

### 1. Get the skills

<details>
<summary><strong>Claude Code</strong></summary>

```
/plugin marketplace add Yis-company/design-skills
/plugin install design-skills@yitam
```

Run `/plugin marketplace update yitam` to pick up new releases.

</details>

<details>
<summary><strong>Codex, and other agents</strong></summary>

```bash
npx skills@latest add Yis-company/design-skills
```

Pick the skills you want, and which coding agents to install them on. **The installer lets you choose which skills to take: make sure `setup-design-skills` is one of them.**

</details>

### 2. Run `/setup-design-skills`

In your agent, run it once per repo. It will:

- Ask you which issue tracker you want to use (GitHub, Linear, or local files)
- Ask you what labels you apply to tickets when you triage them (`/triage` uses labels)
- Ask you where you want to save any docs we create

### 3. You're ready to go.

## Why These Skills Exist

Designers who can read code sit in a useful spot: close enough to the build to see where it drifts from the design, but working mostly in a design tool. These skills target the places that handover usually breaks.

### #1: The Build Didn't Match What I Meant

> "No-one knows exactly what they want"
>
> David Thomas & Andrew Hunt, [The Pragmatic Programmer](https://www.amazon.co.uk/Pragmatic-Programmer-Anniversary-Journey-Mastery/dp/B0833F1T3V)

**The Problem**. A frame shows one state of one screen. Engineers (and their agents) fill in everything else: the empty state, the error, the long name that overflows. The gaps get filled by guesswork.

**The Fix** is a **grilling session**, where the agent asks you detailed questions until the design is decided, and then a design spec that writes those decisions down:

- [`/grill-me`](./skills/productivity/grill-me/SKILL.md): for use outside a repo
- [`/grill-with-docs`](./skills/engineering/grill-with-docs/SKILL.md): same as [`/grill-me`](./skills/productivity/grill-me/SKILL.md), but it also builds a shared language (see below)
- [`/to-spec`](./skills/engineering/to-spec/SKILL.md) and [`/to-tickets`](./skills/engineering/to-tickets/SKILL.md): turn the decisions into a spec of flows, screen states and components, then into tickets engineers can build

### #2: Everyone Calls It Something Different

> With a ubiquitous language, conversations among developers and expressions of the code are all derived from the same domain model.
>
> Eric Evans, [Domain-Driven-Design](https://www.amazon.co.uk/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215)

**The Problem**: The design file says "Card / Elevated", the code says `<Panel raised>`, and the agent invents a third name. Every handover pays for the translation.

**The Fix** is a shared language: a `GLOSSARY.md` that holds the product's terms and the design system's, built up by [`/grill-with-docs`](./skills/engineering/grill-with-docs/SKILL.md) as you go. The [`design-system`](./skills/engineering/design-system/SKILL.md) skill gives the agent the vocabulary for tokens, components and variants.

### #3: The Build Drifted From the Design

**The Problem**: Even a well-specified build drifts: a hex value instead of a token, a near-copy of an existing component, a missing focus state. Each one is small, and they add up.

**The Fix** is checking the code rather than only the screenshot:

- [`design-qa`](./skills/engineering/design-qa/SKILL.md) reviews a branch on two axes: **Fidelity** (does it match the spec and the design, every state included?) and **System** (does it use the design system?). Each finding names the file and gives expected vs actual, so an engineer can act on it.
- [`/audit-design-system`](./skills/engineering/audit-design-system/SKILL.md) surveys the whole codebase for drift every so often and hands you the candidates worth fixing.

## Reference

These split on one axis: who can invoke them. **User-invoked** skills are reachable only when you type them (e.g. `/grill-me`); their job is to orchestrate. **Model-invoked** skills can be invoked by you _or_ reached for automatically by the agent when the task fits; they hold the reusable discipline. A user-invoked skill may invoke model-invoked skills, but never another user-invoked one.

### Engineering

Daily skills for designers who read the codebase: from a design idea to a spec engineers build from, then design QA on what they built.

**User-invoked**

- **[ask-design](./skills/engineering/ask-design/SKILL.md)**: Ask which skill or flow fits your situation. A router over the user-invoked skills in this repo.
- **[grill-with-docs](./skills/engineering/grill-with-docs/SKILL.md)**: Grilling session that also builds your project's domain model, sharpening terminology and updating `GLOSSARY.md` and ADRs inline.
- **[triage](./skills/engineering/triage/SKILL.md)**: Move issues through a state machine of triage roles.
- **[audit-design-system](./skills/engineering/audit-design-system/SKILL.md)**: Scan the codebase for drift from the design system (hardcoded values, near-duplicate components, one-off variants), present it as a visual HTML report, then grill through whichever candidate you pick.
- **[setup-design-skills](./skills/engineering/setup-design-skills/SKILL.md)**: Configure this repo for the design skills (issue tracker, triage labels, domain doc layout, design source). Run once per repo.
- **[to-spec](./skills/engineering/to-spec/SKILL.md)**: Turn the current conversation into a design spec (flows, screen states, components and tokens, accessibility) and publish it to the issue tracker.
- **[to-tickets](./skills/engineering/to-tickets/SKILL.md)**: Break a design spec or conversation into tickets engineers can build, one flow or screen per ticket, each declaring its blocking edges, whether as text in a local file or as native blocking links on a real tracker.
- **[wayfinder](./skills/engineering/wayfinder/SKILL.md)**: Plan a huge chunk of work (more than one agent session can hold) as a shared map of decision tickets on the issue tracker, resolved one at a time until the way to the destination is clear.
- **[retro](./skills/engineering/retro/SKILL.md)**: Suggest improvements to the coding agent's environment (navigation, automated checks, coding standards, steering files, tooling) after a session, most severe first.

**Model-invoked**

- **[prototype](./skills/engineering/prototype/SKILL.md)**: Build a throwaway prototype you run locally to answer a design question: a single shareable HTML file for state/logic, or several toggleable UI variations.
- **[diagnosing-bugs](./skills/engineering/diagnosing-bugs/SKILL.md)**: Disciplined diagnosis loop for hard bugs and performance regressions: build a feedback loop that goes red on this bug → minimise → hypothesise → instrument → fix → regression-test.
- **[research](./skills/engineering/research/SKILL.md)**: Investigate a question against high-trust primary sources and capture the findings as a cited Markdown file in the repo, run as a background agent.
- **[domain-modeling](./skills/engineering/domain-modeling/SKILL.md)**: Actively build and sharpen a project's domain model by challenging terms, stress-testing with scenarios, and updating `GLOSSARY.md` and ADRs inline.
- **[design-system](./skills/engineering/design-system/SKILL.md)**: Shared vocabulary for the design system: tokens, components, variants, slots, drift, and the principles for keeping the code and the design tool in step.
- **[design-qa](./skills/engineering/design-qa/SKILL.md)**: Two-axis review of a build since a fixed point: **Fidelity** (does it match the design spec and the design source, every screen state included?) and **System** (does it use the design system's tokens and components, plus a design-smell baseline?), run as parallel sub-agents.
- **[pr](./skills/engineering/pr/SKILL.md)**: The shape a pull request body should take: a summary as the smallest visual that makes the change clear, before/after evidence that it works, and a merge-danger call (one-way or two-way door, plus blast radius).
- **[wizard](./skills/engineering/wizard/SKILL.md)**: Generate an interactive bash wizard that walks a human through steps only they can perform: provisioning infrastructure, setting up credentials or CI secrets, walking an unfamiliar third-party dashboard, or running a one-off migration or cutover.

### Productivity

General workflow tools, not code-specific.

**User-invoked**

- **[grill-me](./skills/productivity/grill-me/SKILL.md)**: Get relentlessly interviewed about a plan or design until every branch of the design tree is resolved.
- **[handoff](./skills/productivity/handoff/SKILL.md)**: Compact the current conversation into a handoff document so another agent can continue the work.
- **[teach](./skills/productivity/teach/SKILL.md)**: Teach the user a new skill or concept over multiple sessions, using the current directory as a stateful teaching workspace.
- **[to-questionnaire](./skills/productivity/to-questionnaire/SKILL.md)**: Turn a decision you can't answer alone into a Markdown questionnaire for the one person who can, filled in async, or together over a meeting. It grills you about the send (who it's for, what you need back), not the subject.
- **[wait-what](./skills/productivity/wait-what/SKILL.md)**: Fire this the moment a message doesn't land. The agent re-pitches it with the context you're missing, in plain English, using your `GLOSSARY.md` vocabulary.

**Model-invoked**

- **[grilling](./skills/productivity/grilling/SKILL.md)**: Interview the user relentlessly about a plan, decision, or idea until every branch of the design tree is resolved. The reusable interview primitive behind `grill-me`, `grill-with-docs`, `triage`, `wayfinder` and `audit-design-system`.
- **[writing-for-agents](./skills/productivity/writing-for-agents/SKILL.md)**: Writing documents for agents: skills, AGENTS.md/CLAUDE.md, and any doc an agent reaches by a pointer.
