# Seiraiyu Skills

Productivity skills for Claude Code.

## Installation

Via the Seiraiyu Marketplace:

```bash
/plugin marketplace add stonelyd/seiraiyu-marketplace
/plugin install seiraiyu-skills@seiraiyu-marketplace
```

## Skills

### respond-to-coderabbitai

Automatically resolve PR review comments from CodeRabbit (or any reviewer) with atomic commits.

**Features:**
- Discovers all unresolved review threads via GraphQL
- Groups related comments by logical issue
- Creates atomic commits per issue
- Posts threaded replies with commit links
- Handles impasses with escalation to GitHub issues

**Usage:**
```
/seiraiyu-skills:respond-to-coderabbitai
```

## License

MIT
