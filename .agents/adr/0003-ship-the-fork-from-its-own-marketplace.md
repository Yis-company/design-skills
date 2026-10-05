# Ship the fork from its own marketplace

Supersedes the install route in [ADR-0002](./0002-ship-as-a-claude-code-plugin.md). The reasons for shipping a Claude Code plugin at all still hold.

This repo is a fork of `mattpocock/skills`, rewritten for designers who read the codebase but work in a design tool. Its listing in Claude Code's official marketplace belongs to the upstream plugin (`mattpocock-skills`), not to this one. So the fork ships as `design-skills` from the single-plugin marketplace that `.claude-plugin/marketplace.json` already defines (named `yitam`): `/plugin marketplace add Yis-company/design-skills`, then `/plugin install design-skills@yitam`.

Updates don't arrive automatically the way they do from the official marketplace. Users run `/plugin marketplace update yitam`. A listing in the official marketplace can replace this later without changing the plugin itself.

The docs pages move with it: `aihero.dev/skills-<name>` belongs to upstream, so each page is read on GitHub at its `docs/<bucket>/<name>.md` path.
