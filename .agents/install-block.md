# The canonical install block

One install story, one wording. `README.md` and `.changeset/*` must say **this** and nothing else. Change it here first, then propagate.

`design-skills` is not in Claude Code's official marketplace. `.claude-plugin/marketplace.json` makes this repo its own single-plugin marketplace (named `yitam`), so installing is two steps: add the marketplace, then install from it. Why the fork ships this way lives in [ADR-0003](./adr/0003-ship-the-fork-from-its-own-marketplace.md).

## Claude Code: the plugin

<canonical-block name="claude-code">

```
/plugin marketplace add Yis-company/design-skills
/plugin install design-skills@yitam
```

Run `/plugin marketplace update yitam` to pick up new releases.

</canonical-block>

## Codex, and other agents: skills.sh

The plugin is Claude Code only. Everywhere else, [skills.sh](https://skills.sh/Yis-company/design-skills) copies editable skill files into the project. Use the whole-set form on `README.md`:

<canonical-block name="skills-sh-whole-set">

```bash
npx skills@latest add Yis-company/design-skills
```

Pick the skills you want, and which coding agents to install them on. **The installer lets you choose which skills to take: make sure `setup-design-skills` is one of them.**

</canonical-block>

…and the single-skill form wherever one skill is named on its own. **`docs/` pages are not a consumer of this block**: install commands live in the README only. See [writing-docs.md](./writing-docs.md).

<canonical-block name="skills-sh-one-skill">

```bash
npx skills@latest add Yis-company/design-skills --skill=<name>
```

```bash
npx skills@latest update <name>
```

</canonical-block>

`skills@latest` is the pinned spelling in all three.

## The two routes are exclusive

The plugin is a managed, read-only bundle you subscribe to. skills.sh writes files you own and edit. Installing both leaves the user with every skill twice: always say "pick one".
