---
name: design-qa
description: "Check a build against its design along two axes: Fidelity (does it match the design spec and design source, in every screen state the spec lists?) and System (does it use the design system's tokens and components?). Runs both reviews in parallel sub-agents and reports them side by side, with expected vs actual for each finding. Use when the user wants to check a build, branch, PR or work in progress against the design, run design QA, compare the app to the designs, or asks to \"QA since X\"."
---

Two-axis design QA of the diff between `HEAD` and a fixed point the user supplies:

- **Fidelity**: does the build match the design spec and the design source, including every screen state the spec lists?
- **System**: does the build use the design system?

Both axes run as **parallel sub-agents** so they don't pollute each other's context, then this skill aggregates their findings. Call the Skill tool with "design-system" for the vocabulary (token tiers, component, variant, slot, drift, one-off) and use those terms in every finding.

The issue tracker should have been provided to you. If `docs/agents/issue-tracker.md` is missing, tell the user to run `/setup-design-skills`. If `docs/agents/design-source.md` exists, read it for where designs live, how to get exports, and where the token and component sources are.

Read-only: never edit production code. Never call a design-tool MCP. The design source is a link plus exported frames; if you can't open a link, ask the user for exports.

## Process

### 1. Pin the fixed point

Whatever the user said is the fixed point (a commit SHA, branch name, tag, `main`, `HEAD~5`, etc.). If they didn't specify one, ask for it.

Capture the diff command once: `git diff <fixed-point>...HEAD` (three-dot, so the comparison is against the merge-base). Also note the list of commits via `git log <fixed-point>..HEAD --oneline`.

Before going further, confirm the fixed point resolves (`git rev-parse <fixed-point>`) and the diff is non-empty. A bad ref or empty diff should fail here, not inside two parallel sub-agents.

### 2. Identify the spec and design source

Look for the originating design spec, in this order:

1. Issue references in the commit messages (`#123`, `Closes #45`, GitLab `!67`, etc.), fetched via the workflow in `docs/agents/issue-tracker.md`.
2. A path the user passed as an argument.
3. A spec file under `docs/`, `specs/`, or `.scratch/` matching the branch name or feature.
4. If nothing is found, ask the user where the spec is.

From the spec, collect the design-source links, the exported frames, and the list of screen states to check (its **Screen states** and **How design QA will check it** sections, where present). If there is neither a spec nor any export, the **Fidelity** sub-agent skips and reports "no design to check against".

### 3. Identify the system sources

Anything in the repo that documents or defines the design system: design-system docs, token files, the component library (paths from `docs/agents/design-source.md` if present, otherwise search for them).

On top of whatever the repo documents, the System axis always carries the **design-smell baseline** below. Two rules bind it:

- **The repo overrides.** A documented repo standard always wins; where it endorses something the baseline would flag, suppress the smell.
- **Always a judgement call.** Each smell is a labelled heuristic ("possible Near-Duplicate Component"), never a hard violation. Skip anything a linter already enforces.

Each smell reads *what it is* → *how to fix*; match it against the diff:

- **Hardcoded Value**: a raw colour, size, radius, shadow or duration where a token exists. → use the semantic token; if none fits, raise it as a missing token.
- **Off-Scale Spacing or Type**: a spacing or type value that sits between steps of the scale. → snap it to the nearest step the design source uses.
- **Near-Duplicate Component**: a new component that mostly repeats an existing one. → use the existing component, adding a variant or slot if two uses justify it.
- **Overridden Variant**: a component used with style overrides that recreate a variant it lacks, or undo one it has. → use the variant, or propose the missing one.
- **Missing State**: a screen state the spec lists (empty, loading, error, partial, overflow) with no handling in the diff. → build the state as designed, or flag it if the spec left it to the engineer.
- **Missing Focus State**: an interactive element whose focus style is removed or never set. → use the system's focus token or style.
- **Unlabelled Control**: an icon button, input or image with no accessible name. → add the label the spec's copy gives, or ask for one.
- **Low Contrast or Small Target**: text or icon contrast below WCAG AA, or a touch target below the system's minimum. → use a token pair that passes, or enlarge the hit area.
- **Copy Drift**: visible text that differs from the spec's final copy (wording, case, punctuation). → use the spec's copy; flag placeholder text that shipped.
- **Breakpoint Break**: layout that overflows, clips or reflows differently from the design at a breakpoint the spec lists. → follow the spec's responsive rule for that breakpoint.

### 4. Capture the build

If the app runs locally, start it (or use the one already running), reach each screen state the spec lists, and screenshot it at the breakpoints the spec names. Pair every screenshot with its design export, and save the pairs to the OS temp directory so nothing lands in the repo.

If the app doesn't run, or a state can't be reached, review from the diff alone and say so in the report, naming the states that went unseen.

### 5. Spawn both sub-agents in parallel

Every finding on either axis must name the file and hunk and give **expected vs actual**, so an engineer can act on it without asking.

**Fidelity sub-agent prompt** should include:

- The diff command and commit list.
- The spec (path or fetched contents), the design-source links, and the screenshot/export pairs from step 4 (or a note that there are none).
- The brief: "Report: (a) screen states or flow steps the spec asked for that are missing or partial; (b) places the build differs visibly from the design export (layout, hierarchy, copy, imagery); (c) behaviour or UI the spec didn't ask for (scope creep). For each, cite the spec line or export, then expected vs actual, then the file and hunk responsible. Under 400 words."

**System sub-agent prompt** should include:

- The diff command and commit list.
- The system-source paths from step 3, **plus the design-smell baseline from step 3** pasted in full (the sub-agent has no other access to it).
- The brief: "Report, per file and hunk, (a) every place the diff breaks a documented design-system rule: cite the rule (file + the rule); and (b) any baseline smell you spot: name it and quote the hunk. Give expected vs actual for each, naming the token, component or variant that should have been used. Documented-rule breaches can be hard; baseline smells are always judgement calls, and a documented repo rule overrides the baseline. Skip anything a linter enforces. Under 400 words."

If there is no design to check against, skip the Fidelity sub-agent and note this in the final report.

### 6. Aggregate

Present the two reports under `## Fidelity` and `## System` headings, verbatim or lightly cleaned, after a one-line note on how the build was captured (screenshots of which states, or diff only). Do **not** merge or rerank findings, because the two axes are deliberately separate (see _Why two axes_).

End with a one-line summary: total findings per axis, and the worst issue _within each axis_ (if any). Don't pick a single winner across axes: that's the reranking the separation exists to prevent.

## Why two axes

A build can pass one axis and fail the other:

- A screen built entirely from system tokens and components that shows the wrong layout or skips the error state → **System pass, Fidelity fail.**
- A screen that matches the export pixel for pixel through hardcoded values and a copied component → **Fidelity pass, System fail.**

Reporting them separately stops one axis from masking the other.
