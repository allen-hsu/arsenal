---
description: Create a git commit with custom conventions
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git add:*), Bash(git commit:*)
---

# Git Commit

## Current Changes

- Status: !`git status --short`
- Staged diff: !`git diff --cached`
- Unstaged diff: !`git diff`

## Commit Message Convention

Use **Conventional Commits** format:

```
<type>(<scope>): <subject>

[optional body]

[optional footer]
```

### Types

- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only
- `style`: Code style (formatting, semicolons, etc.)
- `refactor`: Code refactoring (no feature or fix)
- `perf`: Performance improvement
- `test`: Adding or updating tests
- `chore`: Maintenance tasks (deps, configs, etc.)

### Rules

1. Subject line: max 50 characters, imperative mood, no period
2. Body: wrap at 72 characters, explain "what" and "why"
3. Scope: optional, describes affected area (e.g., `auth`, `api`, `ui`)

### Examples

```
feat(auth): add OAuth2 login support

fix: resolve race condition in data fetcher

docs(readme): update installation instructions

refactor(api): simplify error handling logic
```

## Task

Based on the changes above:
1. Stage all relevant files (exclude any sensitive files)
2. Create a commit with a message following the convention above
3. Do NOT push to remote
