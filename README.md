# Arsenal

[繁體中文](README.zh-TW.md)

A Claude Code plugin marketplace containing custom skills and commands for AI-assisted development.

## Installation

Add this marketplace to your Claude Code settings:

```bash
claude mcp add-marketplace arsenal https://github.com/allen-hsu/arsenal
```

Or manually add to `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": [
    {
      "name": "arsenal",
      "source": "git:allen-hsu/arsenal"
    }
  ]
}
```

Then install individual plugins:

```bash
claude plugins install arsenal:tech-lead
claude plugins install arsenal:git-commands
claude plugins install arsenal:react-native-mobile
```

## Available Plugins

### tech-lead

Tech lead tools for writing technical specifications and planning features.

**Skills:**
- `tech-spec-writer` - Create comprehensive technical specification documents through interactive Q&A

### git-commands

Custom git workflow commands with conventional commits.

**Commands:**
- `/commit` - Create a git commit with custom conventions

### react-native-mobile

Complete React Native + Expo mobile development toolkit.

**Skills:**
- `react-native-mobile-dev` - App architecture, state management, navigation, and best practices
- `react-native-mobile-design` - UI patterns, design systems, animations, and theming
- `react-native-mobile-devops` - EAS Build, Submit, Update, and CI/CD workflows

**Commands:**
- `/eas-build` - Build your app with EAS
- `/eas-deploy` - Deploy your app to stores
- `/eas-workflow` - Manage EAS Workflows

## Creating New Plugins

### Directory Structure

```
plugins/
└── your-plugin/
    ├── skills/
    │   └── your-skill/
    │       ├── SKILL.md
    │       └── references/
    │           └── guide.md
    └── commands/
        └── your-command.md
```

### Skill Format

Create `SKILL.md` with YAML frontmatter:

```yaml
---
name: skill-name
description: Description including trigger conditions. Use when (1)..., (2)...
---

# Skill Content

Your skill instructions here...
```

### Command Format

Create a `.md` file with YAML frontmatter:

```yaml
---
description: What the command does
allowed-tools: Bash(git status:*), Bash(git diff:*)
---

# Command Instructions

Your command instructions here...
```

### Register in marketplace.json

Add your plugin to `.claude-plugin/marketplace.json`:

```json
{
  "name": "your-plugin",
  "description": "What your plugin does",
  "source": "./plugins/your-plugin",
  "skills": ["./skills/your-skill"],
  "commands": ["./commands/your-command.md"]
}
```

## License

MIT
