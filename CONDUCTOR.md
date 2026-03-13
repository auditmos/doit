---
github:
  owner: auditmos
  repo: doit
  token: $GITHUB_TOKEN

labels:
  todo: "conductor:todo"
  in_progress: "conductor:in-progress"
  review: "conductor:review"
  rework: "conductor:rework"
  done: "conductor:done"
  afk: "conductor:afk"

branch:
  pattern: "conductor/{{number}}-{{slug}}"

workspace:
  root: ./workspaces
  after_clone:
    - "pnpm run setup"

agent:
  command: claude
  max_turns: 25
  retry_budget: 3
  allowed_tools: "Edit,Write,Bash(*)"
  timeout_minutes: 45
  max_cost_per_issue: 10.0

validate:
  commands:
    - "pnpm run lint"

qa:
  enabled: false

pr:
  draft: false
  base_branch: main
  labels:
    - conductor
  reviewers:
    - tkowalczyk

polling:
  interval_ms: 15000
  backoff_max_ms: 120000

sequencing:
  wait_for_merge: true
---

You are implementing issue #{{ issue.number }}: {{ issue.title }}.

## Issue description

{{ issue.body }}

## Parent PRD context

{{ prd.body }}

## Acceptance criteria

{{ acceptance_criteria }}

## User stories

{{ user_stories }}

## Project context

This is a monorepo built on the `auditmos/saas-on-cf` template:

- **Frontend**: TanStack Start (`apps/user-application`) — SSR on Cloudflare Workers
- **Backend**: Hono API (`apps/data-service`) — REST on Cloudflare Workers
- **Shared DB layer**: Drizzle ORM + Neon Postgres (`packages/data-ops`) — schemas, queries, Better Auth
- **Auth**: Better Auth (admin only; client pages are public, accessed via unique slug)
- **AI**: Claude API (conversation only, no tool use) — called from data-service worker
- **Styling**: Tailwind CSS
- **Linting**: Biome (tabs, 100 line width, no unused imports)

## Development commands

```bash
pnpm run setup                    # install deps + build data-ops
pnpm run dev:user-application     # frontend dev (port 3000)
pnpm run dev:data-service         # API dev (port 8788)
pnpm run lint                     # biome check
pnpm run lint:fix                 # biome auto-fix
```

## Rules

1. Read the `.claude/CLAUDE.md` files (root and per-package) before writing any code — they contain project-specific patterns and conventions.
2. Follow the existing code patterns in the saas-on-cf template. Check existing files for import style, error handling, and API patterns before writing new code.
3. Use Drizzle ORM for all database operations. Define schemas in `packages/data-ops/src/drizzle/schema/`.
4. Use Hono for API routes in `apps/data-service/src/routes/`.
5. Use TanStack Start for frontend routes in `apps/user-application/src/routes/`.
6. Run `pnpm run lint:fix` after every file creation or edit.
7. Max 500 lines per source file — split if exceeding.
8. Keep client-facing pages public (no auth). Admin pages use Better Auth.
9. Store AI system prompts in config/DB, not hardcoded in source.
10. Do not create documentation files unless explicitly required by the acceptance criteria.
