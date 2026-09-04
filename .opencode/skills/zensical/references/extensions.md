# Zensical Extensions Reference

Full configuration options for Zensical markdown extensions. Add these to `zensical.toml` under `[project.markdown_extensions]`.

## Admonitions

```toml
admonition = {}
pymdownx.details = {}
pymdownx.superfences = {}
```

Enables `!!! type "Title"` and `??? type "Title"` (collapsible) syntax. Content indented by 4 spaces.

### Types

| Type | Usage |
|------|-------|
| `note` | General notes |
| `abstract` | Summaries, abstracts |
| `info` | Informational |
| `tip` | Tips and best practices |
| `success` | Success stories, positive outcomes |
| `question` | Questions, FAQs |
| `warning` | Warnings, cautionary notes |
| `failure` | Failure cases, negative outcomes |
| `danger` | Dangerous operations, security warnings |
| `bug` | Bug reports, known issues |
| `example` | Examples and demonstrations |
| `quote` | Quotations, citations |

### Inline Admonitions

```markdown
!!! info inline end "Right-aligned"
    Content aligned to the right.

!!! info inline "Left-aligned"
    Content aligned to the left.
```

## Code Blocks

```toml
pymdownx.highlight.anchor_linenums = true
pymdownx.highlight.line_spans = "__span"
pymdownx.highlight.pygments_lang_class = true
pymdownx.inlinehilite = {}
pymdownx.snippets = {}
pymdownx.superfences = {}
```

### Options

| Option | Syntax | Example |
|--------|--------|---------|
| Title | `title="name"` | `` ```yaml title="config.yml" `` |
| Line numbers | `linenums="start"` | `` ```bash linenums="1" `` |
| Highlight lines | `hl_lines="1 3-5"` | `` ```py hl_lines="2 3" `` |
| Copy button | `.copy` attribute | `` ``` { .yaml .copy } `` |
| No copy | `.no-copy` attribute | `` ``` { .yaml .no-copy } `` |

### Annotations

Add `content.code.annotate` to theme features:

```toml
[theme]
features = ["content.code.annotate"]
```

Use `# (1)!` in code comments for annotations.

## Content Tabs

```toml
pymdownx.superfences = {}
pymdownx.tabbed.alternate_style = true
```

### Syntax

```markdown
=== "Tab 1"

    Content 1.

=== "Tab 2"

    Content 2.
```

**Important:** Use `===` (triple equals), not markdown tab syntax.

### Linked Tabs

Add to theme features to sync tabs across pages:

```toml
[theme]
features = ["content.tabs.link"]
```

## Inline Code Highlighting

```toml
pymdownx.inlinehilite = {}
```

Use `#!python` prefix for inline code:

```markdown
The `#!python range()` function generates numbers.
```

## Snippets (Embed External Files)

```toml
pymdownx.snippets = {}
```

Embed file content in code blocks:

````markdown
```title="example.yml"
--8<-- "path/to/file.yml"
```
````

## Table of Contents

```toml
toc = {}
```

Add `[TOC]` to a page for auto-generated table of contents.

## Footnotes

```toml
footnotes = {}
```

```markdown
Text with a footnote.[^1]

[^1]: This is the footnote content.
```

## Abbreviations

```toml
abbr = {}
```

```markdown
*[HTML]: HyperText Markup Language
```

## Deflists

```toml
pymdownx.deflist = {}
```

```markdown
Term
:   Definition of the term
```

## Task Lists

```toml
pymdownx.tasklist = {}
```

```markdown
- [x] Completed task
- [ ] Pending task
```

## GitHub Callouts

```toml
pymdownx.quotes.callouts = true
```

```markdown
> [!NOTE]
> GitHub-style callout.

> [!WARNING]
> Warning callout.
```

Maps: `[!NOTE]` → `note`, `[!TIP]` → `tip`, `[!WARNING]` → `warning`, `[!IMPORTANT]` → `important`, `[!CAUTION]` → `caution`.

## Data Tables

Tables use standard Markdown pipe syntax:

```markdown
| Header 1 | Header 2 |
|----------|----------|
| Cell 1   | Cell 2   |
```
