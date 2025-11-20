# Git Hooks

This directory contains git hooks for automatic validation.

## Setup

Enable hooks (one-time per clone):

```bash
git config core.hooksPath .githooks
```

## Hooks

- **pre-commit**: Runs `make test` before allowing commits

## Bypass

When needed (emergency commits):

```bash
git commit --no-verify
```
