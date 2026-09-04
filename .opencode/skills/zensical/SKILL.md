---
name: zensical
description: "Zensical static site generator for documentation. Use when: writing or editing Zensical markdown content, configuring zensical.toml, setting up navigation, using admonitions code blocks or content tabs, debugging Zensical builds, deploying to GitHub Pages, or when the user mentions Zensical, mkdocs, Material for MkDocs, or documentation site generation."
compatibility: Requires Python 3.10+ and zensical (pip install zensical)
---

# Zensical Reference

Zensical is a modern static site generator by the Material for MkDocs team. It uses TOML config (not YAML) and Python Markdown with pymdown-extensions.

Sources: [zensical.org/docs](https://zensical.org/docs/) |
[Specification](https://agentskills.io/specification)

## Config Format

Config lives in `zensical.toml` at project root. **TOML, not YAML.**

```toml
[project]
site_name = "My Site"
site_url = "https://example.github.io/repo/"
repo_url = "https://github.com/user/repo"
repo_name = "user/repo"

[theme]
name = "material"
features = [
  "content.code.copy",
  "navigation.instant",
  "navigation.tabs",
]

[project.markdown_extensions]
admonition = {}
pymdownx.details = {}
pymdownx.superfences = {}
pymdownx.tabbed.alternate_style = true
```

## Extensions to Enable

The following extensions cover all common content patterns. Add to `zensical.toml`:

```toml
[project.markdown_extensions]
admonition = {}
pymdownx.details = {}
pymdownx.superfences = {}
pymdownx.highlight.anchor_linenums = true
pymdownx.highlight.line_spans = "__span"
pymdownx.highlight.pygments_lang_class = true
pymdownx.inlinehilite = {}
pymdownx.snippets = {}
pymdownx.tabbed.alternate_style = true
```

See [references/extensions.md](references/extensions.md) for full extension config options.

## Admonitions

Use `!!!` for static, `???` for collapsible. Content indented by **4 spaces**.

```markdown
!!! note "Custom Title"
    Content here. 4-space indent required.

??? warning "Collapsible"
    This block starts collapsed.

!!! info inline end
    Inline admonition, aligned right.
```

Supported types: `note`, `abstract`, `info`, `tip`, `success`, `question`, `warning`, `failure`, `danger`, `bug`, `example`, `quote`.

## Code Blocks

Add titles with `title="filename"`, line numbers with `linenums`, highlights with `hl_lines`:

````markdown
```yaml title="container.container"
[Container]
Image=docker.io/library/nginx:latest
PublishPort=8080:80
```

```bash linenums="1"
echo "line numbers enabled"
```

```yaml hl_lines="3 4"
key1: value1
key2: value2
highlighted: this line and next
```
````

## Content Tabs

Use `=== "Tab Name"` (triple equals, not markdown tab syntax):

```markdown
=== "Option A"

    Content for option A.

=== "Option B"

    Content for option B.
```

Tabs can nest code blocks, admonitions, and other tabs.

## Linking Between Pages

Use **relative markdown links**, not HTML. Zensical translates them:

```markdown
[See the quadlet docs](../containerization/quadlets.md)
[Back to home](../index.md)
```

Never link to `.html` outputs — Zensical handles the conversion.

## Page Titles

Priority order (highest first):

1. Title in `nav` config
2. Front-matter `title` field
3. First `# heading` in content
4. Base filename (fallback)

**Gotcha:** If nav title differs from h1, Zensical uses the filename as h1 (unlike MkDocs which uses the nav title).

## Navigation

Define in `zensical.toml` under `[project]` as a TOML array of inline tables. Sections nest arrays inside:

```toml
[project]
nav = [
  { "Home" = "index.md" },
  { "Section" = [
    "section/index.md",
    { "Page" = "section/page.md" },
  ] },
]
```

Without `nav`, Zensical auto-generates from file structure alphabetically.

## Front Matter

Optional YAML front matter in pages:

```markdown
---
title: Custom Page Title
description: Page description for SEO
---
```

## Gotchas

- **TOML not YAML**: `zensical.toml` uses TOML syntax. Nav uses `{ "Key" = "value" }` arrays, not `- Key: value`.
- **4-space indent**: Admonition and tab content needs 4 spaces, not 2.
- **`===` not tabs**: Content tabs use `=== "Name"` syntax, not markdown tab extensions.
- **No `README.md` + `index.md` coexistence**: If both exist in a directory, behavior is undefined. Use one or the other.
- **`nav` titles vs h1**: Zensical currently uses filename as h1 if nav title differs from content h1. Include an explicit `# heading` that matches your nav title.
- **Relative links only**: Never use absolute URLs for internal links. Relative links survive site URL changes.
- **Python Markdown indent**: Requires 4 spaces (or tab) for continuation paragraphs in lists and admonitions — not 2 like some other Markdown parsers.

## GitHub Pages Deployment

Standard workflow: push to `main` → GitHub Actions builds → deploys to `gh-pages` branch.

```yaml
name: Docs
on:
  push:
    branches: [main]
    paths:
      - 'docs/**'
      - 'zensical.toml'
permissions:
  contents: write
jobs:
  docs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.x'
      - run: pip install zensical
      - run: zensical build
      - uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./site
```

## Local Preview

```bash
zensical serve
```

Preview at `localhost:8000`. Auto-rebuilds on file changes.
