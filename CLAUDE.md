# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Arsenal is a Claude Code plugin marketplace containing custom skills and commands. It is registered as a plugin marketplace via `.claude-plugin/marketplace.json` and can be installed in Claude Code to extend its capabilities.

## Repository Structure

```
arsenal/
├── .claude-plugin/
│   └── marketplace.json        # Plugin marketplace configuration
├── plugins/                    # Each plugin has its own isolated directory
│   ├── tech-lead/
│   │   └── skills/
│   │       └── tech-spec-writer/
│   ├── git-commands/
│   │   └── commands/
│   │       └── commit.md
│   └── react-native-mobile/
│       ├── skills/
│       │   ├── react-native-mobile-dev/
│       │   ├── react-native-mobile-design/
│       │   └── react-native-mobile-devops/
│       └── commands/
│           ├── eas-build.md
│           ├── eas-deploy.md
│           └── eas-workflow.md
└── CLAUDE.md
```

## Plugin Development

### Why Isolated Plugin Directories?

Each plugin must have its own directory under `plugins/` to prevent Claude Code from incorrectly loading skills/commands from other plugins. When plugins share the same source directory, the cache includes all files, causing cross-contamination.

### Plugin Structure

Each plugin lives in `plugins/<plugin-name>/` and can contain:
- `skills/` - Directory containing skill subdirectories
- `commands/` - Directory containing command `.md` files

### Skill Structure

Each skill requires:
- `SKILL.md` - Main skill file with YAML frontmatter (`name`, `description`) and markdown content
- `references/` - Optional directory for supporting files (templates, guides)

### Skill Frontmatter Format

```yaml
---
name: skill-name
description: Description including trigger conditions. Use when (1)..., (2)..., (3)...
---
```

### Command Structure

Commands are single `.md` files with YAML frontmatter:

```yaml
---
description: What the command does
allowed-tools: Bash(git status:*), Bash(git diff:*)
---
```

### Registering in marketplace.json

```json
{
  "name": "plugin-name",
  "source": "./plugins/plugin-name",
  "skills": ["./skills/skill-name"],
  "commands": ["./commands/command-name.md"]
}
```

**Important:** The `source` path must point to the plugin's isolated directory, not the repository root.

## External Marketplaces

This project references official Anthropic plugin marketplaces:
- `claude-plugins-official` (anthropics/claude-plugins-official)
- `anthropic-agent-skills` (anthropics/skills)

These are configured in `.claude/settings.json` under `extraKnownMarketplaces`.
