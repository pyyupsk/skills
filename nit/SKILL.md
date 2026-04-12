---
name: nit
description: Use when setting up Git hooks in a JavaScript/TypeScript project, configuring pre-commit or commit-msg hooks, running staged-file hooks, or managing hook sync in CI with @pyyupsk/nit.
license: MIT
metadata:
  author: pyyupsk
  version: "1.1"
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

Add a `"nit"` key to `package.json`. Each hook value is either a plain command string or a `stages` object.

### Plain command

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

### Staged hooks

Run commands only against files matching a glob. Use `{staged_files}` as a placeholder — nit replaces it with the matched staged files before running.

```json
{
  "nit": {
    "hooks": {
      "pre-commit": {
        "stages": {
          "**/*.{ts,js}": "biome check --write {staged_files}",
          "**/*.css": "stylelint --fix {staged_files}"
        }
      },
      "pre-push": "bun typecheck && bun test && bun run build"
    }
  }
}
```

- A stage is skipped entirely if no staged files match its pattern
- Commands without `{staged_files}` run as-is (useful for project-wide checks)

Any valid [Git hook name](https://git-scm.com/docs/githooks) is supported as a key.

## CLI

| Command           | Description                                    |
| ----------------- | ---------------------------------------------- |
| `nit install`     | Write hooks to `.git/hooks/` (default)         |
| `nit sync`        | Alias for `install`                            |
| `nit check`       | Exit 1 if hooks differ from config — CI-safe   |
| `nit exec <hook>` | Run staged hook against currently staged files |

## CI Usage

Use `nit check` in CI to enforce hooks stay in sync:

```sh
nit check   # exits 1 if any hook is missing or has changed
```
