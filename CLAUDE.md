# seiraiyu-skills

Claude Code plugin: skills in `skills/*/SKILL.md` (each reachable as `/seiraiyu-skills:<name>`), published through the seiraiyu-marketplace.

## Gotchas

- `seiraiyu-marketplace`, `seiraiyu-superwisdom`, and `superwisdom-db` are git submodules. Edit them here (they're the canonical clones), and remember submodule changes need their own commit plus a pointer bump in this repo.
- Releasing a skill change means bumping three places: the skill's `metadata.version` in its SKILL.md frontmatter, `version` in `.claude-plugin/plugin.json`, and the `seiraiyu-skills` entry in `seiraiyu-marketplace/.claude-plugin/marketplace.json`.
- Don't add `commands/<name>.md` wrappers for skills. A same-named command expands first and marks the name loaded, so the real SKILL.md body is then suppressed as "previously loaded". Skills are already slash-invocable; put `argument-hint` in the SKILL.md frontmatter.
- Keep SKILL.md files lean: the core workflow inline, long reference material in `skills/<name>/references/`. State each rule once; trust the model's judgment rather than repeating warnings.
