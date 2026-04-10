---
name: hono-api
description: Use when building Hono API servers, defining routes, setting up Hono RPC clients, or deploying to Cloudflare Workers with wrangler.
license: MIT
metadata:
  author: pyyupsk
  version: "1.0"
---

# Hono API

Type-safe API layer with Hono, optimized for Cloudflare Workers and Hono RPC.

## Route Structure

One file per resource in `src/routes/`. Chain methods for type inference:

```ts
// src/routes/users.ts
import { Hono } from "hono";
import { safe } from "@/utils/safe";

const users = new Hono()
  .get("/", async (c) => {
    const [err, data] = await safe(db.query.users.findMany());
    if (err) return c.json({ error: err.message }, 500);
    return c.json(data);
  })
  .get("/:id", async (c) => {
    const [err, user] = await safe(
      db.query.users.findFirst({ where: eq(users.id, c.req.param("id")) }),
    );
    if (err) return c.json({ error: err.message }, 500);
    if (!user) return c.json({ error: "Not found" }, 404);
    return c.json(user);
  });

export default users;
```

## App Composition

Export `AppType` for end-to-end type safety:

```ts
// src/index.ts
import { Hono } from "hono";
import users from "./routes/users";

const app = new Hono().route("/users", users);

export default app;
export type AppType = typeof app;
```

## Hono RPC Client

```ts
import { hc } from "hono/client";
import type { AppType } from "../api/src/index";

export const client = hc<AppType>(process.env.API_URL!);
```

## Cloudflare Workers

### wrangler.jsonc

```jsonc
{
  "$schema": "./node_modules/wrangler/config-schema.json",
  "name": "my-api",
  "main": "src/index.ts",
  "compatibility_date": "2025-01-01",
  "compatibility_flags": ["nodejs_compat"],
}
```

### Bindings — always generated, never manual

```bash
bun wrangler types   # generates worker-configuration.d.ts
```

Use the generated `Env` type with Hono — never define a manual `Bindings` type:

```ts
const app = new Hono<{ Bindings: Env }>();

app.get("/", (c) => {
  const db = c.env.DB;
  return c.json({ env: c.env.ENVIRONMENT });
});
```

## Error Handling

Always use the `safe()` tuple — no bare try/catch in route handlers:

```ts
const [err, data] = await safe(someAsyncCall());
if (err) return c.json({ error: err.message }, 500);
return c.json(data);
```

## Common Mistakes

- Not chaining route methods (breaks `AppType` inference)
- Manually defining `Bindings` type — run `wrangler types` instead
- Using `.then()` chains instead of `async/await` in handlers
- Creating db/client instances per-request — use lazy singletons
