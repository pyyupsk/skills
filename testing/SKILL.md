---
name: testing
description: Use when writing tests, configuring Vitest or Playwright, setting up test file structure, or choosing between unit and e2e test strategies.
license: MIT
metadata:
  author: pyyupsk
  version: "1.0"
---

# Testing

Vitest for unit and integration tests. Playwright for e2e and browser rendering.

## Tooling

| Tool           | Use for                                                      |
| -------------- | ------------------------------------------------------------ |
| **Vitest**     | Unit tests, integration tests, anything that runs in Node    |
| **Playwright** | E2E tests, browser rendering, scenarios Vitest cannot handle |

## Structure

Mirror `src/` exactly in `tests/`:

```tree
src/
└── components/
    └── ui/
        └── button.tsx

tests/
└── components/
    └── ui/
        └── button.test.ts
```

- File naming: `*.test.ts` / `*.test.tsx`
- One test file per source file — no flat dumping into `tests/`

## Convention

Always use `describe` + `it` — never bare `test()`:

```ts
import { describe, expect, it } from "vitest";

describe("button", () => {
  it("renders with primary variant", () => {
    // ...
  });

  it("shows loading spinner when loading is true", () => {
    // ...
  });
});
```

- `describe` — groups by component/function/module name
- `it` — describes behavior in plain English ("it renders…", "it shows…")
- Import all helpers from `"vitest"` — never use globals

## Vitest Config

```ts
// vitest.config.ts
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    environment: "node", // or "jsdom" for DOM tests
    globals: false, // explicit imports always
  },
});
```

## When to Use Playwright

Use Playwright when:

- Test requires a real browser environment
- Testing visual rendering or layout
- Testing user flows across multiple pages
- Vitest + jsdom is insufficient (canvas, WebGL, complex browser APIs)

## Common Mistakes

- **Using bare `test()`** — always wrap in `describe` + use `it`
- **Flat test files** — always mirror `src/` structure in `tests/`
- **Mixing unit and e2e** — keep Vitest and Playwright tests separate
- **Using globals** — always import `describe`, `it`, `expect` from `"vitest"`
- **Vague `it` descriptions** — "it works" is not a test description; describe the behavior
