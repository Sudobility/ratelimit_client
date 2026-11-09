# @sudobility/ratelimit_client

React client library for the Rate Limit API with hooks for fetching rate limit config and usage history.

## Installation

This package is published to **GitHub Packages** (not the public npm registry).
Configure the `@sudobility` scope in your project's `.npmrc`:

```
@sudobility:registry=https://npm.pkg.github.com
```

Then install (a GitHub token with `read:packages` must be available to your package manager):

```bash
bun add @sudobility/ratelimit_client
```

### Peer Dependencies

- `react` >= 18.0.0
- `@sudobility/types`
- `@sudobility/di`

## Usage

```tsx
import { RateLimitClient, useRateLimits } from '@sudobility/ratelimit_client';

// Direct client usage
// `networkClient` is a NetworkClient from @sudobility/di.
// Optional `testMode: true` appends `testMode=true` to every request.
const client = new RateLimitClient({ baseUrl, networkClient /*, testMode */ });
const config = await client.getRateLimitsConfig(token, rateLimitUserId);
const history = await client.getRateLimitHistory('hour', token, rateLimitUserId);

// React hook usage (optional third arg: testMode)
const { config, isLoadingConfig, history, refreshConfig, refreshHistory, error } =
  useRateLimits(networkClient, baseUrl /*, testMode */);

await refreshConfig(token, rateLimitUserId);
await refreshHistory('day', token, rateLimitUserId);
```

## API

### RateLimitClient

| Method | Description |
|--------|-------------|
| `getRateLimitsConfig(token, rateLimitUserId)` | Get rate limit config and current usage |
| `getRateLimitHistory(period, token, rateLimitUserId)` | Get usage history (`'hour'` / `'day'` / `'month'`) |

### useRateLimits Hook

Returns: `config`, `isLoadingConfig`, `history`, `isLoadingHistory`, `error`, `refreshConfig`, `refreshHistory`, `clearError`, `reset`.

### Types (from `@sudobility/types`)

`RateLimitsConfigData`, `RateLimitHistoryData`, `RateLimitPeriodType`

### Local Exports

- `QUERY_KEYS` — TanStack Query key factories (`rateLimitsConfig()`, `rateLimitsHistory(periodType)`)
- `FirebaseIdToken` (type), `UseRateLimitsReturn` (type)
- URL/header helpers: `createAuthHeaders`, `createHeaders`, `buildUrl`, `buildQueryString`, `handleApiError`

## Development

```bash
bun install          # Install dependencies (uses Bun, not npm/yarn/pnpm)
bun run build        # Build to dist/ (tsc + ESM import fixup)
bun run build:watch  # Watch mode build
bun run clean        # Remove dist/
bun run test:unit    # Run Vitest tests once
bun run test:watch   # Run Vitest in watch mode
bun run lint         # Run ESLint
bun run lint:fix     # Fix lint issues
bun run typecheck    # TypeScript check (tsc --noEmit)
bun run format       # Format with Prettier
bun run format:check # Check formatting
```

There is no `test` script; use `test:unit`. Pre-commit:

```bash
bun run typecheck && bun run lint && bun run test:unit && bun run build
```

## License

BUSL-1.1
