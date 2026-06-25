# coding skill

Franco's code-style skill for Claude Code. Expressive naming, fluent interfaces, early returns, Laravel-idiomatic PHP, Laravel-style comments, Stripe-style docs.

Everything lives in [SKILL.md](skills/coding/SKILL.md).

## Installation

### As a plugin

Installs through the [fgilio marketplace](https://github.com/fgilio/claude-plugins) and receives updates as the skill evolves:

```
/plugin marketplace add fgilio/claude-plugins
/plugin install coding@fgilio
```

### Manual clone

Clone and symlink into your Claude Code skills directory:

```bash
git clone https://github.com/fgilio/coding-skill.git ~/dev/skills/coding-skill
ln -s ~/dev/skills/coding-skill/skills/coding ~/.claude/skills/coding
```

This path also works with any agent that supports the open [Agent Skills](https://agentskills.io) format. Updates require a manual `git pull`.

Claude Code loads the skill automatically when writing or reviewing code.

## License

[MIT](LICENSE)
