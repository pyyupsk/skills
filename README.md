# skills

Opinionated agent skills for how I build software — codified into reusable instructions for AI coding agents.

Built on the open [Agent Skills](https://agentskills.io/home) spec.

## Structure

```tree
skills/
├── <skill-name>/
│   ├── SKILL.md
│   ├── scripts/
│   ├── references/
│   └── assets/
└── ...
```

## Install

Install a single skill:

```bash
npx skills install github:pyyupsk/skills/<skill-name>
```

Install all skills at once:

```bash
npx skills install github:pyyupsk/skills "*"
```

## Skills

| Skill                           | Description                                            |
| ------------------------------- | ------------------------------------------------------ |
| [`pyyupsk`](./pyyupsk/)         | Opinionated JS/TS conventions and workflow preferences |
| [`vite-env`](./vite-env/)       | Type-safe environment variables for Vite 8             |
| [`drizzle-orm`](./drizzle-orm/) | Drizzle ORM patterns for PostgreSQL                    |
| [`hono-api`](./hono-api/)       | Hono API routes, RPC client, Cloudflare Workers        |
| [`zod-v4`](./zod-v4/)           | Zod v4 schema, validation, and error handling          |

## License

[MIT](./LICENSE) &copy; [pyyupsk](https://github.com/pyyupsk)
