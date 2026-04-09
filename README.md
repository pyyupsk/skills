# skills

Opinionated agent skills for how I build software — codified into reusable instructions for AI coding agents.

Built on the open [Agent Skills](https://agentskills.io/home) spec.

## Structure

```
skills/
├── <skill-name>/
│   ├── SKILL.md
│   ├── scripts/
│   ├── references/
│   └── assets/
└── ...
```

## Install

```bash
npx @anthropic-ai/skills install github:pyyupsk/skills/<skill-name>
```

## Skills

| Skill                   | Description                                            |
| ----------------------- | ------------------------------------------------------ |
| [`pyyupsk`](./pyyupsk/) | Opinionated JS/TS conventions and workflow preferences |

## License

[MIT](./LICENSE) &copy; [pyyupsk](https://github.com/pyyupsk)
