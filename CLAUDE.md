# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ClaudeKit is a Claude Code plugin marketplace containing custom skills. It is registered as a plugin marketplace via `.claude-plugin/marketplace.json` and can be installed in Claude Code to extend its capabilities.

## Repository Structure

```
claudekit/
├── .claude-plugin/
│   └── marketplace.json    # Plugin marketplace configuration
├── skills/                 # Custom Claude Code skills
│   └── tech-spec-writer/   # Tech spec document generation skill
│       ├── SKILL.md        # Skill definition (frontmatter + workflow)
│       └── references/     # Supporting templates and guides
├── commands/               # Custom slash commands (empty)
└── mcp/                    # MCP server configurations (empty)
```

## Skill Development

### Skill Structure

Each skill lives in `skills/<skill-name>/` and requires:
- `SKILL.md` - Main skill file with YAML frontmatter (`name`, `description`) and markdown content defining the workflow
- `references/` - Optional directory for supporting files (templates, guides)

### Skill Frontmatter Format

```yaml
---
name: skill-name
description: Description including trigger conditions. Use when (1)..., (2)..., (3)...
---
```

The description should include specific trigger conditions so Claude Code knows when to invoke the skill.

### Registering Skills

Add skills to `.claude-plugin/marketplace.json` under `plugins[].skills` array:

```json
{
  "skills": ["./skills/skill-name"]
}
```

## External Marketplaces

This project references official Anthropic plugin marketplaces:
- `claude-plugins-official` (anthropics/claude-plugins-official)
- `anthropic-agent-skills` (anthropics/skills)

These are configured in `.claude/settings.json` under `extraKnownMarketplaces`.
