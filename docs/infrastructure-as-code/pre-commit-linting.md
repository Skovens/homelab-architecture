---
title: Pre-commit & Linting
---

# Pre-commit & Linting

## What

Pre-commit hooks enforce code quality before every commit. Linting catches errors before they reach the server.

## How

### Pre-commit Hooks

```yaml title=".pre-commit-config.yaml"
repos:
  - repo: https://github.com/ansible/ansible-lint
    hooks:
      - id: ansible-lint
  - repo: https://github.com/adrienverge/yamllint
    hooks:
      - id: yamllint
  - repo: https://github.com/ibmdb/detect-secrets
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']
```

### Template Linting

A custom script validates Jinja2 templates:

```bash title="scripts/lint-templates.py"
# Renders every .j2 template, then runs bash -n on the output
# Catches syntax errors before deployment
```

### Branch Protection

The `no-commit-to-branch` hook prevents direct commits to `main`:

```yaml
  - repo: https://github.com/pre-commit/pre-commit-hooks
    hooks:
      - id: no-commit-to-branch
        args: ['--branch', 'main']
```

## Why

The lint pipeline catches three categories of errors:

1. **YAML syntax** — malformed files that Ansible would reject at runtime
2. **Ansible best practices** — deprecated modules, missing become, insecure permissions
3. **Secret leaks** — API keys, tokens, or passwords accidentally committed

The template linter is custom because standard YAML linting doesn't catch Jinja2 render errors. A template that renders to invalid bash or invalid systemd is invisible to YAML lint — but the custom script renders every template and validates the output.

!!! tip "Run before commit"
    Pre-commit runs automatically, but you can also run manually:

    ```bash
    pre-commit run --all-files
    ```
