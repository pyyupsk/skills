---
name: nit
description: Use when setting up Git hooks in a JavaScript/TypeScript project, configuring pre-commit or commit-msg hooks, or managing hook sync in CI with @pyyupsk/nit.
license: MIT
metadata:
  author: pyyupsk
  version: "1.0"
---

# nit

Lightweight, zero-dependency Git hooks manager for JS/TS projects. Config lives in `package.json` — no extra files.

## Installation

```sh
bun add -D @pyyupsk/nit
```

Add a `prepare` script so hooks install automatically on `bun install`:

```json
{
  "scripts": {
    "prepare": "nit install"
  }
}
```

## Configuration

Add a `"nit"` key to `package.json`:

```json
{
  "nit": {
    "hooks": {
      "pre-commit": "bun run lint && bun run typecheck",
      "commit-msg": "bun run test"
    }
  }
}
```

Any valid [Git hook name](https://git-scm.com/docs/githooks) is supported as a key. The value is the shell command to run.

## CLI

| Command       | Description                                  |
| ------------- | -------------------------------------------- |
| `nit install` | Write hooks to `.git/hooks/` (default)       |
| `nit sync`    | Alias for `install`                          |
| `nit check`   | Exit 1 if hooks differ from config — CI-safe |

## CI Usage

Use `nit check` in CI to enforce committed hooks stay in sync:

```sh
nit check   # exits 1 if any hook is missing or has changed
```
