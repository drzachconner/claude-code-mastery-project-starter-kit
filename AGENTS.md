# Claude-Code-Mastery-Starter — Agent Context

## What This Project Is

A Claude Code project starter kit based on the Claude Code Mastery Guides V1-V5 by TheDecipherist. It provides a pre-configured scaffold with 26 slash commands, typed APIs, test infrastructure, Docker gates, and database conventions ready to use in new projects.

Source: [github.com/TheDecipherist/claude-code-mastery](https://github.com/TheDecipherist/claude-code-mastery)

## Tech Stack

- **Language**: TypeScript (strict mode) — always, no exceptions
- **Package manager**: pnpm
- **Database**: StrictDB (`strictdb` npm package) — unified wrapper for MongoDB, PostgreSQL, MySQL, SQLite, Elasticsearch. NEVER use native drivers directly.
- **Testing**: Vitest (unit/integration) + Playwright (E2E)
- **Build**: TypeScript compiler (`tsc`)
- **CSS optimization**: Classpresso (post-build, auto-runs as `postbuild`)
- **MCP servers**: ClassMCP (semantic CSS), StrictDB-MCP (database access)

## Architecture

**Multi-service layout with fixed ports:**

| Service | Dev Port | Test Port |
|---------|----------|-----------|
| Website | 3000 | 4000 |
| API | 3001 | 4010 |
| Dashboard | 3002 | 4020 |

**Database access pattern (StrictDB):**
- Create ONE instance at startup: `const db = await StrictDB.create({ uri: process.env.STRICTDB_URI! })`
- Share the instance — never create multiple connections
- NEVER import native drivers (`mongodb`, `pg`, `mysql2`, etc.)
- Ad-hoc/test queries go through `scripts/db-query.ts` system only

**API versioning:** All endpoints must use `/api/v1/` prefix — no exceptions.

## Directory Structure

```
project/
├── CLAUDE.md / AGENTS.md
├── CLAUDE.local.md          # Personal overrides (gitignored)
├── .claude/
│   ├── commands/            # 26 slash commands
│   ├── skills/              # Triggered expertise templates
│   ├── agents/              # Custom subagents
│   └── hooks/               # 9 enforcement hooks
├── project-docs/
│   ├── ARCHITECTURE.md      # System overview
│   ├── INFRASTRUCTURE.md    # Deployment details
│   └── DECISIONS.md         # Architectural decisions
├── src/
│   ├── handlers/            # Business logic
│   ├── adapters/            # External service wrappers
│   └── types/               # Shared TypeScript types
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── scripts/
│   ├── db-query.ts          # Test query master — index of all dev/test queries
│   ├── queries/             # Individual query files (dev/test only)
│   └── build-content.ts     # Markdown → HTML article builder
├── content/                 # Markdown source files for articles
├── docs/
│   └── user-guide.html      # Interactive User Guide
├── claude-mastery-project.conf  # Profile presets for /new-project
├── playwright.config.ts     # E2E test config (test ports 4000/4010/4020)
├── vitest.config.ts
└── tsconfig.json
```

## Critical Rules

1. **TypeScript always** — strict mode, no `any` without documented justification
2. **API versioning** — every endpoint uses `/api/v1/` prefix
3. **StrictDB only** — no native database drivers, no Mongoose/ODMs
4. **Database queries** — all ad-hoc/test queries go through `scripts/queries/<name>.ts` + registry in `scripts/db-query.ts`. Never create standalone query scripts.
5. **Graceful shutdown** — every Node.js entry point must handle `SIGTERM`/`SIGINT`/`uncaughtException`/`unhandledRejection` calling `db.gracefulShutdown()`
6. **Tests** — minimum 3 assertions per test (URL + element visible + data correct). Never write tests without assertions.
7. **Quality gates** — no file > 300 lines, no function > 50 lines
8. **Parallel awaits** — use `Promise.all` for independent async operations, never sequential
9. **No secrets in code** — always use environment variables
10. **Never auto-deploy** — always ask before deploying to production

## Branch Workflow

Auto-branch hook is ON — commits to `main` are blocked. Always branch before editing:

```bash
git branch --show-current   # check current branch
git checkout -b feat/<name> # create feature branch BEFORE editing
```

Branch naming: `feat/`, `fix/`, `docs/`, `refactor/`, `chore/`, `test/`

## Key Commands

```bash
pnpm dev              # Start dev server
pnpm build            # Type-check + compile
pnpm typecheck        # TypeScript only, no emit
pnpm test             # All tests (unit + E2E)
pnpm test:unit        # Vitest unit/integration tests
pnpm test:e2e         # Playwright E2E (kills test ports first)
pnpm db:query <name>  # Run a registered dev/test query
pnpm content:build    # Build Markdown → HTML
```

## Service Ports — Fixed, Never Change

- Kill ports before starting: `lsof -ti:3000,3001,3002 | xargs kill -9 2>/dev/null`
- Always specify port explicitly: `npx next dev -p 3002`

## Command Scope Classification

Commands have `scope:` in YAML frontmatter:
- **`scope: project`** (16 commands) — copied to scaffolded projects via `/new-project`
- **`scope: starter-kit`** (10 commands) — kit management only, never copied

## Documentation Sync (Mandatory)

When updating any feature, keep all these in sync: `README.md`, `docs/index.html`, `project-docs/`, `CLAUDE.md` quick reference, `tests/STARTER-KIT-VERIFICATION.md`, inline comments, test descriptions.

When adding a command or hook — also update `.claude/settings.json` to wire hooks.

## What NOT to Touch

- **Port assignments** — 3000/3001/3002 (dev) and 4000/4010/4020 (test) are fixed
- **`scripts/queries/` outside the db-query system** — never create standalone query files in `src/`
- **Native database drivers** — `mongodb`, `pg`, `mysql2`, `mssql`, `better-sqlite3`, `mongoose` are forbidden
- **`.env` file** — never commit; use `.env.example` for templates
- **`main` branch directly** — always use feature branches

## Go / Python Support

The starter kit supports multiple languages detected by profile or `go.mod`/`pyproject.toml`:
- **Go**: `cmd/` + `internal/` layout, `golangci-lint`, table-driven tests, `context.Context` on all I/O
- **Python**: type hints required, `async def` for FastAPI I/O, pytest only, `.venv/` virtual env, Pydantic models

## Related

- See `CLAUDE.md` for Claude Code-specific configuration
