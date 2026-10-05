# Engineering

Daily skills for designers who read the codebase: from a design idea to a spec engineers build from, then design QA on what they built.

## User-invoked

Reachable only when you type them (Claude Code: `disable-model-invocation: true`; Codex: `policy.allow_implicit_invocation: false` in `agents/openai.yaml`).

- **[ask-design](./ask-design/SKILL.md)**: Ask which skill or flow fits your situation. A router over the user-invoked skills in this repo.
- **[grill-with-docs](./grill-with-docs/SKILL.md)**: Grilling session that also builds your project's domain model, sharpening terminology and updating `GLOSSARY.md` and ADRs inline.
- **[triage](./triage/SKILL.md)**: Move issues through a state machine of triage roles.
- **[audit-design-system](./audit-design-system/SKILL.md)**: Scan the codebase for drift from the design system (hardcoded values, near-duplicate components, one-off variants), present it as a visual HTML report, then grill through whichever candidate you pick.
- **[setup-design-skills](./setup-design-skills/SKILL.md)**: Configure this repo for the design skills (issue tracker, triage labels, domain doc layout, design source). Run once per repo.
- **[to-spec](./to-spec/SKILL.md)**: Turn the current conversation into a design spec (flows, screen states, components and tokens, accessibility) and publish it to the issue tracker.
- **[to-tickets](./to-tickets/SKILL.md)**: Break a design spec or conversation into tickets engineers can build, one flow or screen per ticket, each declaring its blocking edges, whether as text in a local file or as native blocking links on a real tracker.
- **[wayfinder](./wayfinder/SKILL.md)**: Plan a huge chunk of work (more than one agent session can hold) as a shared map of decision tickets on the issue tracker, resolved one at a time until the way to the destination is clear.
- **[retro](./retro/SKILL.md)**: Suggest improvements to the coding agent's environment (navigation, automated checks, coding standards, steering files, tooling) after a session, most severe first.

## Model-invoked

Model- or user-reachable (rich trigger phrasing so the model can reach for them).

- **[prototype](./prototype/SKILL.md)**: Build a throwaway prototype you run locally to answer a design question: a single shareable HTML file for state/logic, or several toggleable UI variations.

- **[diagnosing-bugs](./diagnosing-bugs/SKILL.md)**: Disciplined diagnosis loop for hard bugs and performance regressions: build a feedback loop that goes red on this bug → minimise → hypothesise → instrument → fix → regression-test.
- **[research](./research/SKILL.md)**: Investigate a question against high-trust primary sources and capture the findings as a cited Markdown file in the repo, run as a background agent.
- **[domain-modeling](./domain-modeling/SKILL.md)**: Actively build and sharpen a project's domain model by challenging terms, stress-testing with scenarios, and updating `GLOSSARY.md` and ADRs inline.
- **[design-system](./design-system/SKILL.md)**: Shared vocabulary for the design system: tokens, components, variants, slots, drift, and the principles for keeping the code and the design tool in step.
- **[design-qa](./design-qa/SKILL.md)**: Two-axis review of a build since a fixed point: **Fidelity** (does it match the design spec and the design source, every screen state included?) and **System** (does it use the design system's tokens and components, plus a design-smell baseline?), run as parallel sub-agents.
- **[pr](./pr/SKILL.md)**: The shape a pull request body should take: a summary as the smallest visual that makes the change clear, before/after evidence that it works, and a merge-danger call (one-way or two-way door, plus blast radius).
- **[wizard](./wizard/SKILL.md)**: Generate an interactive bash wizard that walks a human through steps only they can perform: provisioning infrastructure, setting up credentials or CI secrets, walking an unfamiliar third-party dashboard, or running a one-off migration or cutover.
