# RateLimit Client

> **Git policy — never auto-commit or auto-push.** Leave your work in the working tree.
> Run `git commit`, `git push`, or `gh pr create` **only when the user explicitly asks in
> that turn**. Approval for an earlier change does not carry forward, and finishing a task
> is not permission to commit it.

React client library for Rate Limit API with hooks for fetching rate limit config and history.

**npm**: `@sudobility/ratelimit_client`

## Tech Stack

- **Language**: TypeScript
- **Runtime**: Bun
- **Package Manager**: Bun (do not use npm/yarn/pnpm for installing dependencies)
- **Build**: TypeScript compiler (`tsc`) + `scripts/fix-esm-imports.mjs` (rewrites relative imports to `.js` for ESM)
- **Test**: Vitest (jsdom, globals enabled via `vitest.config.ts`)
- **Registry**: Published to GitHub Packages (`@sudobility` scope → `npm.pkg.github.com`, see `.npmrc`)

## Project Structure

```
src/
├── index.ts              # Public exports
├── types.ts              # Client-specific types
├── hooks/                # React hooks
│   ├── index.ts          # Hook exports
│   └── useRateLimits.ts  # Rate limits hook
├── network/              # HTTP client utilities
│   ├── index.ts
│   └── RateLimitClient.ts # API client class
├── utils/                # Utility functions
│   ├── index.ts
│   └── ratelimit-helpers.ts # Helper functions
└── test/
    └── setupDom.ts       # jsdom test setup helpers
```

Tests are colocated (`*.test.ts`) and excluded from the build via `tsconfig.build.json`.

## Commands

```bash
bun install          # Install deps (Bun only)
bun run build        # Build to dist/
bun run build:watch  # Watch mode build
bun run clean        # Remove dist/
bun run test:unit    # Run Vitest once  (NOTE: there is no `test` script)
bun run test:watch   # Run Vitest in watch mode
bun run lint         # Run ESLint
bun run lint:fix     # Fix lint issues
bun run typecheck    # TypeScript check (tsc --noEmit)
bun run format       # Format with Prettier
bun run format:check # Check formatting
```

## API Client

### RateLimitClient
```typescript
import { RateLimitClient } from '@sudobility/ratelimit_client';

// testMode?: true appends `testMode=true` to every request URL
const client = new RateLimitClient({ baseUrl, networkClient, testMode: false });

// Get rate limit config and current usage
// rateLimitUserId is required (e.g., entity slug, user ID)
const config = await client.getRateLimitsConfig(token, rateLimitUserId);

// Get usage history by period
const history = await client.getRateLimitHistory('hour', token, rateLimitUserId);
const history = await client.getRateLimitHistory('day', token, rateLimitUserId);
const history = await client.getRateLimitHistory('month', token, rateLimitUserId);
```

## Hooks

### useRateLimits
```typescript
import { useRateLimits } from '@sudobility/ratelimit_client';

const {
  config,           // Current rate limit config
  isLoadingConfig,  // Loading state for config
  history,          // Usage history data
  isLoadingHistory, // Loading state for history
  error,            // Error message
  refreshConfig,    // Function to refresh config
  refreshHistory,   // Function to refresh history
  clearError,       // Clear error state
  reset,            // Reset all state
} = useRateLimits(networkClient, baseUrl, /* testMode */ false);

// Fetch config (rateLimitUserId is required)
await refreshConfig(token, rateLimitUserId);

// Fetch history (rateLimitUserId is required)
await refreshHistory('hour', token, rateLimitUserId);
await refreshHistory('day', token, rateLimitUserId);
```

## API Endpoints

The `rateLimitUserId` is a generic identifier that can be an entity slug, user ID, or any other identifier depending on the application.

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/ratelimits/:rateLimitUserId` | Get rate limit config and current usage |
| GET | `/api/v1/ratelimits/:rateLimitUserId/history/:periodType` | Get usage history (hour/day/month) |

## Types

Types are imported from `@sudobility/types`:

```typescript
import type {
  RateLimitsConfigData,
  RateLimitHistoryData,
  RateLimitPeriodType,
} from '@sudobility/types';
```

## Peer Dependencies

Required in consuming app:
- `react` >= 18.0.0
- `@sudobility/types` - Common types
- `@sudobility/di` - NetworkClient interface

## Publishing

Published to **GitHub Packages** (`@sudobility` scope). CI (`.github/workflows/ci-cd.yml`)
auto-publishes on push to `main`/`develop` via the reusable `johnqh/workflows` unified-cicd
workflow with `publish-target: github`. `prepublishOnly` runs `clean` + `build` automatically.

```bash
bun run build  # or `bun run prepublishOnly` (clean + build)
npm publish    # publishes to npm.pkg.github.com per .npmrc; CI normally does this
```

## Architecture

```
ratelimit_client (this package)
    ↑
ratelimit_pages (page containers)
    ↑
shapeshyft_app (frontend)
sudojo_app (frontend)
```

## Workspace Context & Downstream Impact

Part of the **ShapeShyft** multi-project workspace under `../`. Consumers: `ratelimit_pages`
(direct, uses `useRateLimits`) and `shapeshyft_app` (direct + transitive via `ratelimit_pages`).
After publishing, run `bun update @sudobility/ratelimit_client` + rebuild in each consumer.

## Local Dev Workflow

```bash
# In this project:
bun link

# In ratelimit_pages:
bun link @sudobility/ratelimit_client

# Rebuild after changes:
bun run build

# When done, unlink:
bun unlink @sudobility/ratelimit_client && bun install
```

## Pre-Commit Checklist

No `verify` script. Run checks manually:

```bash
bun run typecheck && bun run lint && bun run test:unit && bun run build
```

Note: `test:unit` runs Vitest once (`vitest run`), not watch mode.

## Gotchas

- **`rateLimitUserId` is a generic identifier** -- it can be an entity slug, user ID, or any identifier. Do not assume it is always a Firebase UID.
- **Tests require jsdom environment** -- the test script passes `--environment jsdom`. Tests run in a browser-like environment.
- **Types come from `@sudobility/types`, not this package** -- `RateLimitsConfigData`, `RateLimitHistoryData`, etc. are in the shared types package.
- **`testMode`** -- both `RateLimitClient` and `useRateLimits` take an optional `testMode` flag that appends `testMode=true` to every request URL for server-side sandbox behavior.
- **ESM import fixup** -- `bun run build` runs `scripts/fix-esm-imports.mjs` after `tsc` to rewrite relative imports to explicit `.js` extensions. Do not hand-edit `dist/`; the import rewrite is required for the package to resolve as pure ESM.
- **No parent `CLAUDE.md`** -- there is currently no `../CLAUDE.md`; do not rely on one.

## Git Workflow

- Do not use feature branches for code changes. Always stay on the current branch.
