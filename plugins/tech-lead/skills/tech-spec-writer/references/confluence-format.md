# Confluence Wiki Markup Format Guide

Convert Markdown to Confluence wiki markup for direct pasting.

## How to Paste in Confluence

1. Open Confluence page editor
2. Press `Cmd+Shift+D` (Mac) or `Ctrl+Shift+D` (Windows) to open markup editor
3. Paste the wiki markup content
4. Press the shortcut again to return to visual editor

---

## Conversion Reference

### Headings

| Markdown | Confluence |
|----------|------------|
| `# H1` | `h1. H1` |
| `## H2` | `h2. H2` |
| `### H3` | `h3. H3` |

### Text Formatting

| Markdown | Confluence |
|----------|------------|
| `**bold**` | `*bold*` |
| `*italic*` | `_italic_` |
| `~~strikethrough~~` | `-strikethrough-` |
| `` `code` `` | `{{code}}` |

### Lists

**Unordered:**
```
Markdown:          Confluence:
- Item 1           * Item 1
- Item 2           * Item 2
  - Nested         ** Nested
```

**Ordered:**
```
Markdown:          Confluence:
1. First           # First
2. Second          # Second
   1. Nested       ## Nested
```

### Links

| Markdown | Confluence |
|----------|------------|
| `[text](url)` | `[text\|url]` |
| `[text](#anchor)` | `[text\|#anchor]` |

### Tables

**Markdown:**
```markdown
| Header 1 | Header 2 |
|----------|----------|
| Cell 1   | Cell 2   |
```

**Confluence:**
```
||Header 1||Header 2||
|Cell 1|Cell 2|
```

### Code Blocks

**Markdown:**
````markdown
```javascript
const x = 1;
```
````

**Confluence:**
```
{code:javascript}
const x = 1;
{code}
```

Supported languages: `javascript`, `typescript`, `python`, `sql`, `bash`, `json`, `java`, `go`, `rust`

### Info Panels

**Confluence-specific panels:**

```
{info}
This is an info panel.
{info}

{note}
This is a note/warning panel.
{note}

{warning}
This is a warning panel.
{warning}

{tip}
This is a tip panel.
{tip}
```

### Status Macros

```
{status:colour=Green|title=DONE}
{status:colour=Yellow|title=IN PROGRESS}
{status:colour=Red|title=BLOCKED}
{status:colour=Blue|title=TODO}
```

### Expandable Sections

```
{expand:Click to expand}
Hidden content here.
{expand}
```

### Table of Contents

```
{toc}
```

Place at the top of the document for auto-generated TOC.

---

## Full Example

**Markdown Input:**
```markdown
## Problem Statement

Users **cannot** track their portfolio. This causes:

- Manual calculations
- Errors in reporting

| Metric | Current | Target |
|--------|---------|--------|
| Errors | 100/day | <10/day |

```sql
SELECT * FROM portfolios;
```
```

**Confluence Output:**
```
h2. Problem Statement

Users *cannot* track their portfolio. This causes:

* Manual calculations
* Errors in reporting

||Metric||Current||Target||
|Errors|100/day|<10/day|

{code:sql}
SELECT * FROM portfolios;
{code}
```

---

## Tips

1. **Tables**: Confluence tables don't need alignment dashes
2. **Nested lists**: Use `**` for level 2, `***` for level 3
3. **Images**: Use `!image.png!` or `!image.png|width=500!`
4. **Mentions**: Use `[~username]` to mention users
5. **Jira links**: Just paste Jira URLs - Confluence auto-links them
