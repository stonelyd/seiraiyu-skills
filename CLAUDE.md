# seiraiyu-skills

Claude Code plugin: skills in `skills/*/SKILL.md`, slash commands in `commands/*.md`, published through the seiraiyu-marketplace.

## Gotchas

- `seiraiyu-marketplace`, `seiraiyu-superwisdom`, and `superwisdom-db` are git submodules. Edit them here (they're the canonical clones), and remember submodule changes need their own commit plus a pointer bump in this repo.
- Releasing a skill change means bumping three places: the skill's `metadata.version` in its SKILL.md frontmatter, `version` in `.claude-plugin/plugin.json`, and the `seiraiyu-skills` entry in `seiraiyu-marketplace/.claude-plugin/marketplace.json`.
- `commands/*.md` are one-line wrappers that invoke the same-named skill — behavior belongs in the SKILL.md, never in the command file.
- Keep SKILL.md files lean: the core workflow inline, long reference material in `skills/<name>/references/`. State each rule once; trust the model's judgment rather than repeating warnings.
