# Arsenal

[English](README.md)

一個 Claude Code 插件市集，包含自訂的 skills 和 commands，用於 AI 輔助開發。

## 安裝

將此市集加入 Claude Code 設定：

```bash
claude mcp add-marketplace arsenal https://github.com/allen-hsu/arsenal
```

或手動加入 `.claude/settings.json`：

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

接著安裝個別插件：

```bash
claude plugins install arsenal:tech-lead
claude plugins install arsenal:git-commands
claude plugins install arsenal:react-native-mobile
```

## 可用插件

### tech-lead

技術主管工具，用於撰寫技術規格書和規劃功能。

**Skills：**
- `tech-spec-writer` - 透過互動式問答建立完整的技術規格文件

### git-commands

自訂 git 工作流程指令，支援 conventional commits。

**Commands：**
- `/commit` - 使用自訂規範建立 git commit

### react-native-mobile

完整的 React Native + Expo 行動應用開發工具組。

**Skills：**
- `react-native-mobile-dev` - 應用程式架構、狀態管理、導航和最佳實踐
- `react-native-mobile-design` - UI 模式、設計系統、動畫和主題設定
- `react-native-mobile-devops` - EAS Build、Submit、Update 和 CI/CD 工作流程

**Commands：**
- `/eas-build` - 使用 EAS 建置應用程式
- `/eas-deploy` - 部署應用程式到商店
- `/eas-workflow` - 管理 EAS Workflows

## 建立新插件

### 目錄結構

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

### Skill 格式

建立 `SKILL.md`，包含 YAML frontmatter：

```yaml
---
name: skill-name
description: 描述觸發條件。Use when (1)..., (2)...
---

# Skill 內容

你的 skill 指令...
```

### Command 格式

建立 `.md` 檔案，包含 YAML frontmatter：

```yaml
---
description: 這個指令做什麼
allowed-tools: Bash(git status:*), Bash(git diff:*)
---

# Command 指令

你的 command 指令...
```

### 註冊到 marketplace.json

將插件加入 `.claude-plugin/marketplace.json`：

```json
{
  "name": "your-plugin",
  "description": "你的插件功能描述",
  "source": "./plugins/your-plugin",
  "skills": ["./skills/your-skill"],
  "commands": ["./commands/your-command.md"]
}
```

## 授權

MIT
