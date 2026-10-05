## What it does

`design-qa` checks a build against its design. It reads the diff between `HEAD` and a fixed point you name (a commit, a branch, a tag, `main`, `HEAD~5`) and reviews it along two axes. **Fidelity** asks whether the build matches the design [spec](https://www.aihero.dev/ai-coding-dictionary/spec) and the design source, in every screen state the spec lists. **System** asks whether it uses the design system's tokens and components. Each axis runs in its own [sub-agent](https://www.aihero.dev/ai-coding-dictionary/subagent) so neither sees the other's reasoning.

The skill never merges or re-ranks the two axes. The report ends with a worst issue *per axis* and declines to name a single winner across them. A build can pass one axis and fail the other. A screen built entirely from system components that skips the error state passes System and fails Fidelity. A screen that matches the export pixel for pixel through hardcoded values and a copied component does the reverse. A blended verdict lets the passing axis hide the failing one.

## When to reach for it

Type `/design-qa`, or the agent reaches for it automatically when you ask to check a build, a branch, a PR or work in progress against the design.

| Your situation | Reach for |
| --- | --- |
| Engineers have built something and you want to know if it matches the design *and* uses the system | `design-qa` |
| Nothing is built yet and engineers need to know what to build | [to-spec](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/to-spec.md) |
| The whole codebase has drifted from the system, not one diff | [audit-design-system](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/audit-design-system.md) |
| Something is broken and you do not know why | [diagnosing-bugs](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/diagnosing-bugs.md) |

You must supply the fixed point. If you do not, the skill asks for one rather than guessing. Before it spawns anything, it checks that the ref resolves and that the diff is not empty, so a mistyped branch name fails in front of you instead of inside two sub-agents.

## Prerequisites

The System axis needs nothing. It reads whatever the repo documents (design-system docs, token files, the component library) and falls back on a built-in baseline when the repo documents nothing.

The Fidelity axis needs a design to check against. It looks for the spec in this order:

1. Issue references in the commit messages (`#123`, `Closes #45`, a GitLab `!67`), fetched through `docs/agents/issue-tracker.md`.
2. A path you pass in as an argument.
3. A spec file under `docs/`, `specs/`, or `.scratch/` matching the branch or feature name.
4. Asking you.

Step 1 depends on `docs/agents/issue-tracker.md`, which [setup-design-skills](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/setup-design-skills.md) writes. The same setup can write `docs/agents/design-source.md`, which tells the skill where designs live, how to get exports, and where the token and component sources are. Without it the skill still works, it just searches more. With no spec and no export at all, the skill skips the Fidelity sub-agent and says so rather than inventing a design.

## The two axes

| | Fidelity | System |
| --- | --- | --- |
| Question | Is it the design? | Is it built from the system? |
| Reads | The design spec, the design-source exports, screenshots of the build | Design-system docs, token files, the component library, plus the design-smell baseline |
| Reports | Missing screen states, visible differences from the export, scope creep | Documented rule breaches (can be hard), and smells (always judgement calls) |
| Every finding gives | The spec line or export, expected vs actual, the file and hunk | The rule or smell, expected vs actual, the file and hunk |

Expected vs actual plus a file and hunk is the bar for every finding, so an engineer can act on it without coming back to ask what you meant.

Before the sub-agents run, the skill **captures the build**: if the app runs locally, it screenshots each screen state the spec lists and pairs every screenshot with its design export. If the app does not run, it reviews from the diff alone and the report says which states went unseen.

The **design-smell baseline** sits under the repo's own rules: Hardcoded Value, Off-Scale Spacing or Type, Near-Duplicate Component, Overridden Variant, Missing State, Missing Focus State, Unlabelled Control, Low Contrast or Small Target, Copy Drift, Breakpoint Break. Each is a labelled heuristic ("possible Overridden Variant"), never a hard violation, and each states what the smell is and how to fix it. The repo's own documentation is the [primary source](https://www.aihero.dev/ai-coding-dictionary/primary-source) on the System axis, and **the repo always overrides** the baseline.

## Common questions

**What happened to the two-axis code review skill?**

This skill replaced it. The two-axis shape is the same; the axes changed. Spec became Fidelity, which compares the build to the design spec and its exports. Standards became System, which checks tokens and components against a design-smell baseline instead of a list of code smells.

**Does it need the app running?**

No, but it is much better with it. With the app running it compares screenshots of each screen state to the matching export. Without it, the Fidelity axis can only reason from the diff about layout and copy, and the report says so.

**Can it open my design-tool link?**

Only if the link is publicly readable as a page or image. The skill never connects to a design tool directly. If it cannot open a link, it asks you for exported frames, so have exports of each screen state ready.

**Its sub-agents keep invoking it again and spawn more agents.**

This was a known bug in the skill this one replaced, and the sub-agent briefs here have the same shape, so watch for it. Nothing in the briefs forbids delegation, so a sub-agent can find the skill again and fan out again. The fix people applied on forks is one line appended to both briefs: "Do not invoke `/design-qa` or spawn additional agents: perform this review directly." If you run it unattended, watch the agent count.

**After every ticket, or once at the end?**

Both work, and the skill does not decide for you. Per ticket keeps each diff small enough that the Fidelity axis has one flow or screen to check, which matches how [to-tickets](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/to-tickets.md) slices work. A pass at the end of a branch catches screens that drifted apart from each other. If you are unsure, check per ticket and run one final pass against the branch point.

**Can I trust the findings?**

Not without checking. Sub-agent output is a hypothesis, not evidence. The skill combines the two reports as they are, or lightly cleaned. It does not re-verify each claim against the files, so a finding can cite the wrong hunk or overstate a difference. Read the expected vs actual on each finding, and look at the screenshot pair where there is one, before you send it to an engineer.

**Why does it find new problems every single time I run it?**

Each fix adds new code to review, and the judgement-call half of the System axis gives different results from run to run. There is no convergence guarantee. Treat a pass as a list of leads. Act on the ones with a cited rule or a visible difference behind them, then stop. Do not run it in a loop until it comes back clean, because it never will.

**Does it check uncommitted work?**

No. It diffs `<fixed-point>...HEAD`. The three-dot form measures from the merge-base and excludes staged and working-tree changes. Ask for a commit first, then run it.

## It's working if

- It refuses to start on a bad ref or an empty diff, before any sub-agent is spawned.
- The report opens by saying how the build was captured: which states were screenshotted, or that it worked from the diff alone.
- The report arrives as two separate blocks under `## Fidelity` and `## System`, not one merged list.
- Every finding gives expected vs actual and names a file and hunk, so you can forward it to an engineer without rewriting it.
- The closing summary gives a worst issue per axis and declines to pick an overall winner.
- With no spec or export available, the Fidelity block says so instead of listing a design it inferred from the code.

## Where it fits

`design-qa` is the check at the tail of the designer flow: `grill-with-docs → to-spec → to-tickets → engineers build → design-qa`. It also stands alone on any branch or PR you point it at.

- [to-spec](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/to-spec.md) writes the screen states and the "How design QA will check it" section the Fidelity axis reads, so a vague spec makes that axis vague.
- [design-system](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/design-system.md) supplies the words (token tiers, variant, slot, drift) every finding uses.
- [audit-design-system](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/audit-design-system.md) is the whole-codebase counterpart, because this skill only looks at one diff.

[ask-design](https://github.com/Yis-company/design-skills/blob/main/docs/engineering/ask-design.md) routes across the whole set when you are unsure which skill the situation wants.
