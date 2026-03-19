# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**portless** is a CLI tool that replaces port numbers with stable, named `.localhost` URLs for local development. It's a reverse proxy mapping named hostnames (e.g., `myapp.localhost:1355`) to dynamically-assigned ports on `127.0.0.1`. Published as an npm global CLI tool (`npm install -g portless`).

## Commands

```bash
pnpm install          # Install dependencies
pnpm build            # Build all (via turbo)
pnpm test             # Unit tests (vitest, via turbo)
pnpm test:e2e         # E2E tests across 11 frameworks (via turbo)
pnpm test:coverage    # Unit tests with coverage
pnpm test:watch       # Watch mode for unit tests
pnpm lint             # ESLint
pnpm lint:fix         # ESLint with auto-fix
pnpm typecheck        # TypeScript type-checking
pnpm format           # Prettier format
pnpm format:check     # Prettier check
pnpm dev              # Watch mode build
```

Run a single test file: `pnpm --filter portless exec vitest run src/routes.test.ts`

## Architecture

**Monorepo** (pnpm workspaces + Turborepo). Only `packages/portless/` is published to npm. Everything else is private.

- `packages/portless/` -- The npm package. ESM-only, zero runtime deps (except `chalk`). Built with tsup.
- `apps/docs/` -- Next.js 16 docs site (Tailwind CSS v4, MDX, AI SDK chat)
- `tests/e2e/` -- Vitest e2e tests with fixtures for 11 frameworks
- `examples/google-oauth/` -- Example: Next.js + NextAuth + Google OAuth with custom TLD
- `skills/` -- Claude Code agent skills

### Core Package Internals (`packages/portless/src/`)

- `cli.ts` -- Single-file CLI entry point with all commands (~1500 lines). This is the main file to modify for CLI changes.
- `proxy.ts` -- Dual HTTP/1.1 + HTTP/2 reverse proxy server with byte-peeking protocol detection and SNI for per-hostname TLS certs.
- `routes.ts` -- RouteStore: file-backed JSON route management with directory-based locking.
- `certs.ts` -- TLS certificate generation (root CA + per-hostname SNI certificates).
- `hosts.ts` -- /etc/hosts sync and cleanup.
- `auto.ts` -- Project name inference, git worktree detection.
- `types.ts` -- RouteInfo and ProxyServerOptions interfaces.

### Key Patterns

- **ESM-only**: `"type": "module"`, `import`/`export`, `.js` extensions in imports
- **Node.js 20+**: Uses `node:` protocol imports throughout
- **File-based state**: Routes stored as JSON in `~/.portless` (or `/tmp/portless`), concurrency via `mkdir`-based file locking
- **Version injection**: `__VERSION__` defined at build time via tsup from `package.json`, declared in `cli.ts` as `declare const __VERSION__: string`
- **Cross-platform**: Explicit Windows/Unix branching for paths, commands, permissions

## Conventions

- **Package manager**: Always `pnpm`. Exception: end-user install docs use `npm install -g`.
- **No emojis** anywhere in code, comments, output, or docs.
- **Boolean env vars**: Document as `0`/`1` only (not `true`/`false`) in help, docs, and README.
- **Docs sync**: Changes affecting CLI behavior must update `README.md`, `skills/portless/SKILL.md`, and `cli.ts` `--help` output simultaneously.
- **Unused variables**: Prefix with `_` (enforced by ESLint).
- **Prettier**: Semicolons, double quotes, trailing commas (es5), 100 print width, 2-space indent.
- **Changesets**: Only `packages/portless/` is versioned/published. Docs, e2e, and examples are private and ignored.
- **Reserved CLI names**: `run`, `get`, `alias`, `hosts`, `list`, `trust`, `proxy` cannot be used as app names.
