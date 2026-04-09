---
name: pyyupsk
description: Opinionated JS/TS conventions and workflow preferences by pyyupsk. Use when setting up projects, configuring linting, or establishing coding standards.
license: MIT
metadata:
  author: pyyupsk
  version: "1.0"
---

# Core Conventions

## File Organization

- Organize by feature or sub-category, not by file type
- Separate types into dedicated `types.ts` or `types/*.ts` files
- Prefer isomorphic code (works in Node, browser, workers)

## TypeScript

- Explicit return types on exported functions
- Fix type errors in implementation first — never widen types or add casts as a shortcut
- Comments should explain "why", not "how"

## Linting & Formatting

- **Biome** for JS/TS linting and formatting
- **Prettier** for everything else (Markdown, YAML, JSON, etc.)

## Package Manager

Use **bun** for installing packages and running scripts:

```bash
bun add <package>
bun run <script>
```

## package.json

Always sort `scripts` alphabetically (A-Z) when adding or modifying entries.

## Git Workflow

### Commits

Follow [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) strictly:

```text
<type>[optional scope]: <description>
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `chore`, `ci`

### Branch Naming

Name branches after the content of the changes, not the process:

```text
feat/user-profile-sync     ✓
simplify/code-cleanup      ✗
```

### Worktrees Over Stash

Prefer `git worktree` over `git stash`. Worktrees provide isolated working directories without risking lost or conflicting stashed changes.

### Pre-Commit

Run the full suite before every commit:

```bash
bun format; bun check; bun lint; bun typecheck; bun test; bun run build
```

All steps must pass with zero warnings before committing.
