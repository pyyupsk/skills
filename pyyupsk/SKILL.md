---
name: pyyupsk
description: Opinionated JS/TS conventions and workflow preferences by pyyupsk. Use when setting up projects, writing TypeScript/React/Vue components, configuring tooling, or applying coding patterns.
license: MIT
metadata:
  author: pyyupsk
  version: "2.1"
---

# Core Conventions

## Tooling

- **Biome** for JS/TS linting and formatting
- **Prettier** for everything else (Markdown, YAML, JSON)
- **Bun** as runtime and package manager — never `npm`/`yarn`/`pnpm` unless forced
- **`$schema`** in all config files — prefer local `node_modules/` path over HTTPS URL

```json
{ "$schema": "./node_modules/@biomejs/biome/configuration_schema.json" }
```

## Project Structure

All projects use `src/` as root. Files/folders: lowercase single-dash (`some-thing.tsx`). Hooks: camelCase (`useSomething.ts`).

```tree
src/
├── app/route/_components/   # scoped, not shared outside route
├── components/<category>/   # shared, grouped by category (never flat)
├── components/ui/           # shadcn/ui only
├── hooks/useSomething.ts
├── lib/  types/  utils/
└── assets/fonts/index.ts    # export const sans, mono
```

## TypeScript

### Never use `any`

```ts
// ❌
const val = thing as any;
// ✅ find the library type first; last resort only:
const val = thing as unknown as CustomType;
```

### Props

```tsx
// type over interface; Readonly<> always
type ButtonProps = { variant: "primary" | "secondary" };
function Button({ variant }: Readonly<ButtonProps>) {}

// children via PropsWithChildren — never add children manually
function Card({ title, children }: PropsWithChildren<CardProps>) {}

// extending HTML elements
import type { ComponentPropsWithoutRef } from "react";
type ButtonProps = ComponentPropsWithoutRef<"button"> & { variant?: string };
```

### Static values & type utilities

```ts
const VARIANTS = ["primary", "secondary", "ghost"] as const;
type Variant = (typeof VARIANTS)[number]; // extract array item type

// satisfies — type safety without widening
const theme = { primary: "#3b82f6" } satisfies Record<string, string>;

// derive types — never define manually when you can extract
type FilterState = ReturnType<typeof useFilters>;

// extract return type from Convex backend functions
import type { FunctionReturnType } from "convex/server";
import type { api } from "@/convex/_generated/api";
type Category = FunctionReturnType<typeof api.categories.get>;
type CategoryGroup = FunctionReturnType<typeof api.categoryGroups.list>[number];
```

### Modern String / Array / Object Methods

Prefer modern, non-mutating alternatives (Sonar-aligned):

```ts
// ❌
str.replace("foo", "bar")
str.match(/foo(\d+)/)       // use RegExp.prototype.exec instead
arr.sort((a, b) => a.localeCompare(b))
arr.reverse()
arr.splice(1, 1)
arr[0] = "new"
arr[arr.length - 1]
str[str.length - 1]
isNaN(value)
isFinite(value)
obj.hasOwnProperty(key)
JSON.parse(JSON.stringify(obj))

// ✅
str.replaceAll("foo", "bar")
/foo(\d+)/.exec(str)
arr.toSorted((a, b) => a.localeCompare(b))
arr.toReversed()
arr.toSpliced(1, 1)
arr.with(0, "new")
arr.at(-1)
str.at(-1)
Number.isNaN(value)
Number.isFinite(value)
Object.hasOwn(obj, key)
structuredClone(obj)
```

### UUIDs

```ts
// ❌
import { v4 as uuidv4 } from "uuid";

// ✅
crypto.randomUUID();
```

## Naming Conventions

- Handler **props**: `on` prefix — `onSubmit`, `onClick`
- Internal handlers: `handle` prefix — `handleSubmit`, `handleClick`

## Patterns

### Function declarations

```ts
// ❌ arrow functions for functions/components
const fetchUser = async (id: string) => { ... }
const Button = () => { ... }

// ✅ function declarations
async function fetchUser(id: string) { ... }
function Button() { ... }

// ✅ const only for values
const API_URL = "https://api.example.com"
```

### Early returns

```tsx
// ❌ nested ternaries
return loading ? <Spinner /> : error ? <Error /> : <Content />;

// ✅ early returns
if (loading) return <Spinner />;
if (error) return <Error />;
return <Content />;
```

### Error handling — `safe()` tuple

Never throw inside business logic. Return errors as values:

```ts
export async function safe<T>(promise: Promise<T>): Promise<[Error, null] | [null, T]> {
    try {
        return [null, await promise]
    } catch (e) {
        return [e instanceof Error ? e : new Error(String(e)), null]
    }
}

// usage
const [err, user] = await safe(fetchUser(id))
if (err) return <Error message={err.message} />
return <UserCard user={user} />
```

### Async/await — never `.then()`

```ts
// ❌ fetchUser().then(u => console.log(u))
// ✅ const user = await fetchUser()
```

### `cn()` — conditional only

```tsx
// ❌ static string, no cn() needed
<div className="flex items-center gap-2" />

// ✅ conditional or composed
<div className={cn("flex items-center gap-2", isActive && "bg-accent", className)} />
```

### Custom hooks over inline logic

```tsx
// ❌ fetch inside component
// ✅ extract to hook
function useUser(id: string) { ... }
function UserCard() {
    const { user } = useUser(id)
    return <div>{user?.name}</div>
}
```

### No barrel `export *`

```ts
// ❌ export * from "./Button"  // circular deps, breaks tree-shaking
// ✅ import { Button } from "@/components/ui/button"
```

### Singletons for third-party clients

```ts
let client: Resend | undefined;
function getResend() {
  if (!client) client = new Resend(process.env.RESEND_API_KEY!);
  return client;
}
export const resend = getResend();
```

## Next.js

```tsx
// Root layout — suppressHydrationWarning on <html>, fonts on <body> only
export default function RootLayout({ children }: PropsWithChildren) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body className={cn(sans.variable, mono.variable)}>{children}</body>
    </html>
  );
}
// Base styles in CSS: body { @apply min-h-screen bg-background font-sans antialiased; }

// Fonts — src/assets/fonts/index.ts
export const sans = Geist({ subsets: ["latin"], variable: "--font-sans" });

// "use client" — push boundary as deep as possible, never at page level
// Server Actions — for mutations, not API routes
// Metadata — always export metadata/generateMetadata, never <head> tags
```

## Vue (Composition API)

`<script setup lang="ts">` only. Order inside script:

1. `type` / `interface`
2. `defineProps` / `defineEmits`
3. `ref` / `reactive` / `computed`
4. composables / functions
5. lifecycle hooks (`onMounted`, `onUnmounted` last)

## State Management

**Zustand** (React) — one store per domain, file named `useXxxStore.ts` (camelCase, same as hooks):

```ts
const initialState = { search: "", status: "all" as const };
export const useFilterStore = create<FilterStore>((set) => ({
  ...initialState,
  setSearch: (search) => set({ search }),
  reset: () => set(initialState),
}));
```

**Pinia** (Vue) — Option Stores, one per domain, named `useXxxStore`.

## Git

Conventional Commits: `feat`, `fix`, `chore`, `refactor`, `docs`, `style`, `test`, `perf`

Branch names reflect content not process: `feat/user-profile-sync` not `simplify/code-cleanup`

Prefer `git worktree` over `git stash`. Pre-commit: `bun format; bun check; bun lint; bun typecheck; bun test; bun run build` — all must pass.

## package.json

Sort `scripts` alphabetically (A–Z).
