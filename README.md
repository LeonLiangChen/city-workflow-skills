# workflow-skills

Team-shared AI agent skills with the `city-` prefix, compatible with the [skills CLI](https://github.com/vercel-labs/skills).

## Available Skills

| Skill | Description |
|-------|-------------|
| [city-lark-diagram](./skills/city-lark-diagram/) | Generate diagrams from descriptions and upload to Lark whiteboards |
| [city-lark-solution-doc](./skills/city-lark-solution-doc/) | Summarize solution discussions into structured Lark documents with embedded whiteboards |

## Installation

```bash
# Install all skills
npx skills add <repo-url>

# Install a specific skill
npx skills add <repo-url> --skill city-lark-diagram
```

> If the repo is private, clone it locally first, then install from the local path:
>
> ```bash
> git clone <repo-url> workflow-skills
> npx skills add ./workflow-skills
> ```

## Prerequisites

Most skills share the following global dependencies:

- [`lark-cli`](https://github.com/larksuite/lark-cli) — Lark Suite CLI
- [`@larksuite/whiteboard-cli`](https://www.npmjs.com/package/@larksuite/whiteboard-cli) — Whiteboard rendering CLI
- Lark user auth: `lark-cli auth login --domain wiki,docs,drive`

See each skill's `SKILL.md` for its specific dependencies.

## Adding a New Skill

1. Create `skills/<skill-name>/SKILL.md` with the required frontmatter:
   ```markdown
   ---
   name: city-<skill-name>
   description: One-line description of what the skill does.
   ---
   ```
2. Commit and push — teammates install with `npx skills add`.
