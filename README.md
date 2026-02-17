# claude-devs

A Claude Code plugin for managing reusable skills, commands, and development workflow guides.

## Overview

This repository is a [Claude Code plugin](https://docs.anthropic.com/en/docs/claude-code) that bundles custom skills, slash commands, hooks, and reference documents into a single installable package. Use it to extend Claude Code with team-shared or personal development workflows.

## Project Structure

```
claude-devs/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest
├── skills/                  # Agent skills (directory-based, with SKILL.md entry point)
│   └── code-review/
│       ├── SKILL.md         # Skill definition
│       └── reference.md     # Supporting reference docs
├── commands/                # Simple slash commands (.md files)
│   └── quick-check.md
├── agents/                  # Custom subagent definitions
├── hooks/                   # Event hooks
│   └── hooks.json
├── scripts/                 # Utility and hook scripts
├── LICENSE
└── README.md
```

## Quick Start

### Load as a local plugin

```bash
claude --plugin-dir /path/to/claude-devs
```

### Use the bundled skills and commands

```bash
# Full code review
/claude-devs:code-review src/app.ts

# Quick scan
/claude-devs:quick-check src/utils.js
```

### Debug mode

```bash
claude --plugin-dir /path/to/claude-devs --debug
```

## Adding a New Skill

1. Create a directory under `skills/`:
   ```
   skills/my-skill/
   ├── SKILL.md          # Required — YAML frontmatter + instructions
   ├── reference.md      # Optional — detailed reference material
   └── scripts/          # Optional — helper scripts
   ```

2. Define the skill in `SKILL.md`:
   ```yaml
   ---
   name: my-skill
   description: What this skill does
   allowed-tools: Read, Grep
   ---

   Instructions for Claude when this skill is invoked...
   ```

3. Test it:
   ```bash
   claude --plugin-dir .
   /claude-devs:my-skill [args]
   ```

## Adding a New Command

Create a `.md` file in `commands/`:

```yaml
---
description: What this command does
---

Instructions for Claude...
```

Commands are simpler than skills — single file, no supporting documents.

## Contributing

1. Create a branch from `main`.
2. Add your skill or command following the structure above.
3. Test locally with `--plugin-dir`.
4. Submit a pull request.

## License

MIT
