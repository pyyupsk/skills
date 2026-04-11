---
name: nextjs
description: Use when building Next.js pages, layouts, Server Components, Server Actions, configuring fonts, metadata, or environment variables.
license: MIT
metadata:
  author: pyyupsk
  version: "1.0"
---

# Next.js

App Router patterns — Server Components by default, minimal client footprint.

## Root Layout

```tsx
// src/app/layout.tsx
import { sans, mono } from "@/assets/fonts";
import { cn } from "@/lib/utils";
import type { PropsWithChildren } from "react";

export default function RootLayout({ children }: PropsWithChildren) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body className={cn(sans.variable, mono.variable)}>{children}</body>
    </html>
  );
}
```

- `suppressHydrationWarning` on `<html>` — prevents theme provider mismatches
- Font CSS variables on `<body>` only — never add classes to `<html>`
- Base styles in CSS, not className:

```css
body {
  @apply min-h-screen bg-background text-foreground font-sans antialiased;
}
```

## Fonts

Define all fonts in `src/assets/fonts/index.ts`, export as named constants:

```ts
// src/assets/fonts/index.ts
import { Geist, Geist_Mono } from "next/font/google";

export const sans = Geist({ subsets: ["latin"], variable: "--font-sans" });
export const mono = Geist_Mono({ subsets: ["latin"], variable: "--font-mono" });
```

## `"use client"` Boundary

Every component is a Server Component by default. Add `"use client"` only when needed: `useState`, `useEffect`, event handlers, browser APIs. Push the boundary as deep as possible.

```tsx
// ❌ forces entire page tree to client
"use client"
export default function Page() {
  return <div><StaticContent /><InteractiveButton /></div>
}

// ✅ isolate only the interactive part
export default function Page() {
  return <div><StaticContent /><InteractiveButton /></div>
}

// interactive-button.tsx
"use client"
export function InteractiveButton() { ... }
```

## Data Fetching

Fetch directly in Server Components — no API routes needed for internal data:

```tsx
export default async function UserPage({ params }: { params: { id: string } }) {
  const [err, user] = await safe(fetchUser(params.id));
  if (err) return <Error />;
  return <UserCard user={user} />;
}
```

Use React Query or SWR only in Client Components that need real-time updates or optimistic mutations.

## Server Actions

Use Server Actions for mutations — not API routes:

```tsx
async function createPost(formData: FormData) {
  "use server";
  const result = createPostSchema.safeParse(Object.fromEntries(formData));
  if (!result.success) return { error: z.prettifyError(result.error) };
  await db.insert(posts).values(result.data);
}
```

- Always `safeParse` on user input — never `parse`
- Return `{ error }` on failure — never throw inside Server Actions
- No bare `try/catch` — use the `safe()` tuple pattern for async calls

## Metadata

Always define `metadata` or `generateMetadata` — never use `<head>` tags directly:

```tsx
// static
export const metadata: Metadata = {
  title: "Page Title",
  description: "Page description",
};

// dynamic
export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const post = await getPost(params.slug);
  return { title: post.title };
}
```

## Environment Variables

Use `@t3-oss/env-nextjs` (`createEnv`) for build-time Zod validation — never access `process.env` directly:

```ts
// src/env.ts
import { createEnv } from "@t3-oss/env-nextjs";
import { z } from "zod";

export const env = createEnv({
  server: {
    DATABASE_URL: z.string().url(),
    RESEND_API_KEY: z.string().min(1),
  },
  client: {
    NEXT_PUBLIC_APP_URL: z.string().url(),
  },
  runtimeEnv: {
    DATABASE_URL: process.env.DATABASE_URL,
    RESEND_API_KEY: process.env.RESEND_API_KEY,
    NEXT_PUBLIC_APP_URL: process.env.NEXT_PUBLIC_APP_URL,
  },
});
```

## Common Mistakes

- **`"use client"` at page level** — isolate only the interactive leaf component
- **`<head>` tags** — always use `export const metadata` or `generateMetadata`
- **`process.env` directly** — always import from `src/env.ts`
- **API routes for internal data** — fetch directly in Server Components
- **`parse` on user input** — always `safeParse` in Server Actions
- **Base styles in `className`** — put `@apply` rules in CSS, not the layout component
