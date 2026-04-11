---
name: eslint-config
description: Use when setting up @pyyupsk/eslint-config, writing eslint.config.ts, adding Vue or UnoCSS linting, or configuring ESLint rules for TypeScript projects.
license: MIT
metadata:
  author: pyyupsk
  version: "1.0"
---

# @pyyupsk/eslint-config

Personal ESLint flat config built on `typescript-eslint` + `eslint-plugin-unicorn`. Use when Biome doesn't fully support the target (e.g. Vue, special plugins).

## Install

```bash
bun add -d eslint @pyyupsk/eslint-config
```

## Basic Setup

```ts
// eslint.config.ts
import { defineConfig } from "@pyyupsk/eslint-config";

export default defineConfig();
```

## Options

```ts
defineConfig({
  ignores: ["dist/**"], // additional ignore patterns (.gitignore auto-applied)
  perfectionist: true, // import/export sorting — recommended-natural preset
  stylistic: true, // @stylistic formatting (see defaults below)
  vue: true, // Vue flat/recommended + typescript-eslint parser
  vue: { unocss: true }, // Vue + @unocss/eslint-plugin
});
```

## Core Rules (always enabled)

| Rule                                             | Effect                                    |
| ------------------------------------------------ | ----------------------------------------- |
| `@typescript-eslint/consistent-type-definitions` | Enforces `type` over `interface`          |
| `@typescript-eslint/no-explicit-any`             | Bans `any`                                |
| `prefer-object-has-own`                          | `Object.hasOwn` over `obj.hasOwnProperty` |
| `unicorn/prefer-at`                              | `.at(-1)` over `[arr.length - 1]`         |
| `unicorn/prefer-number-properties`               | `Number.isNaN` / `Number.isFinite`        |
| `unicorn/prefer-string-replace-all`              | `replaceAll` over `replace`               |
| `unicorn/prefer-structured-clone`                | `structuredClone` over JSON clone         |

Also includes `tseslint.configs.recommended` and `tseslint.configs.stylistic`.

## Stylistic Defaults (`stylistic: true`)

| Setting      | Value            |
| ------------ | ---------------- |
| Quotes       | Single           |
| Indent       | 2 spaces         |
| Semicolons   | None             |
| Comma dangle | Always multiline |

## Perfectionist (`perfectionist: true`)

Applies `recommended-natural` preset from `eslint-plugin-perfectionist` — sorts imports, exports, and object keys in natural order.

## Vue (`vue: true`)

- `eslint-plugin-vue` flat/recommended
- `typescript-eslint` parser for `<script lang="ts">`
- `vue/multi-word-component-names` turned off

Enable UnoCSS linting alongside Vue:

```ts
defineConfig({ vue: { unocss: true } });
```

## Named Exports

Use individual exports when composing your own config:

```ts
import { core, perfectionist, stylistic, vue } from "@pyyupsk/eslint-config";
import { defineConfig } from "eslint/config";

export default defineConfig([
  ...core({ ignores: ["dist/**"] }),
  ...perfectionist(),
  ...stylistic(),
  ...vue({ unocss: true }),
]);
```

## Common Mistakes

- **Using `interface`** — `consistent-type-definitions` will error; always use `type`
- **Using `any`** — `no-explicit-any` is set to `error`; find the library type or use `unknown`
- **`obj.hasOwnProperty`** — use `Object.hasOwn(obj, key)` instead
- **`arr[arr.length - 1]`** — use `arr.at(-1)` instead
- **`isNaN` / `isFinite`** — use `Number.isNaN` / `Number.isFinite`
- **`str.replace`** — use `str.replaceAll` for string literals
- **`JSON.parse(JSON.stringify(x))`** — use `structuredClone(x)`
