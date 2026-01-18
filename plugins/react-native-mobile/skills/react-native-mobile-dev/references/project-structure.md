# Project Structure

> This structure assumes you created a project with `npx rn-new@latest` using recommended options: Expo Router + NativeWind + i18next. The `src/` directory is added manually for better organization.

## Root Directory

```
├── app/                   # Expo Router screens (file-based routing)
├── src/                   # Application source code
├── assets/                # Static assets (images, fonts)
├── locales/               # i18n translation files
├── app.json               # Expo configuration
├── babel.config.js        # Babel configuration
├── tsconfig.json          # TypeScript configuration
├── tailwind.config.js     # Tailwind/NativeWind configuration
├── metro.config.js        # Metro bundler configuration
├── nativewind-env.d.ts    # NativeWind TypeScript support
├── global.css             # Global styles (NativeWind)
└── package.json
```

## App Directory (Expo Router)

File-based routing with Expo Router:

```
app/
├── _layout.tsx            # Root layout (providers, fonts, etc.)
├── index.tsx              # Home / Entry screen
├── +not-found.tsx         # 404 page
├── (tabs)/                # Tab navigation group
│   ├── _layout.tsx        # Tab bar configuration
│   ├── index.tsx          # First tab
│   ├── explore.tsx        # Explore tab
│   └── settings.tsx       # Settings tab
├── (auth)/                # Auth flow group
│   ├── _layout.tsx
│   ├── login.tsx
│   └── register.tsx
└── [id].tsx               # Dynamic route
```

## Source Directory (src/)

Feature-based architecture with clear separation of concerns:

```
src/
├── core/                      # Core infrastructure (no feature dependencies)
│   ├── config/                # App configuration
│   │   ├── env.ts             # Environment variables (Zod validated)
│   │   └── constants.ts       # App constants
│   │
│   ├── services/              # External service integrations
│   │   ├── api/               # HTTP client
│   │   │   ├── client.ts      # Axios instance
│   │   │   ├── interceptors.ts
│   │   │   └── error-handler.ts
│   │   ├── storage/           # Persistent storage (MMKV)
│   │   │   ├── index.ts
│   │   │   └── keys.ts
│   │   ├── network/           # Network & offline support
│   │   │   ├── monitor.ts     # NetInfo monitoring
│   │   │   ├── queue.ts       # Offline request queue
│   │   │   └── sync.ts        # Sync mechanism
│   │   └── logger/            # Logging service
│   │       └── index.ts
│   │
│   ├── i18n/                  # Internationalization
│   │   ├── index.ts           # i18next configuration
│   │   ├── types.ts           # Type-safe translations
│   │   ├── resources.ts       # Namespace loader
│   │   └── utils.ts           # RTL + language helpers
│   │
│   └── providers/             # Context providers
│       ├── theme-provider.tsx
│       ├── query-provider.tsx
│       ├── i18n-provider.tsx
│       ├── network-provider.tsx
│       └── app-providers.tsx  # Combined provider wrapper
│
├── features/                  # Feature modules (self-contained)
│   ├── auth/                  # Authentication feature
│   │   ├── api/               # Auth API hooks
│   │   │   ├── use-login.ts
│   │   │   ├── use-register.ts
│   │   │   └── use-logout.ts
│   │   ├── components/        # Auth components
│   │   │   ├── screens/       # Screen-level components
│   │   │   │   ├── login-screen.tsx
│   │   │   │   └── register-screen.tsx
│   │   │   └── widgets/       # Feature widgets
│   │   │       ├── login-form.tsx
│   │   │       └── social-login.tsx
│   │   ├── hooks/             # Auth hooks
│   │   │   └── use-auth-state.ts
│   │   ├── store/             # Feature state (optional)
│   │   │   └── auth-form-store.ts
│   │   ├── schemas/           # Zod schemas
│   │   │   └── login-schema.ts
│   │   ├── types/             # Auth types
│   │   │   └── index.ts
│   │   ├── __tests__/         # Feature tests
│   │   │   ├── components/
│   │   │   ├── hooks/
│   │   │   └── api/
│   │   └── index.ts           # Feature barrel export
│   │
│   ├── transactions/          # Example feature module
│   │   ├── api/
│   │   │   ├── use-transactions.ts
│   │   │   └── use-create-transaction.ts
│   │   ├── components/
│   │   │   ├── screens/
│   │   │   └── widgets/
│   │   ├── types/
│   │   ├── __tests__/
│   │   └── index.ts
│   │
│   └── settings/              # Settings feature
│       ├── components/
│       ├── __tests__/
│       └── index.ts
│
├── shared/                    # Shared utilities (cross-feature)
│   ├── components/            # Reusable UI components
│   │   ├── ui/                # Design system primitives
│   │   │   ├── button.tsx
│   │   │   ├── text.tsx
│   │   │   ├── input.tsx
│   │   │   └── index.ts
│   │   ├── forms/             # Form controls
│   │   │   ├── controlled-input.tsx
│   │   │   ├── controlled-select.tsx
│   │   │   └── index.ts
│   │   ├── layout/            # Layout components
│   │   │   ├── container.tsx
│   │   │   ├── safe-area.tsx
│   │   │   └── index.ts
│   │   └── feedback/          # Feedback components
│   │       ├── loading.tsx
│   │       ├── empty-state.tsx
│   │       ├── error-boundary.tsx
│   │       ├── offline-banner.tsx
│   │       └── index.ts
│   ├── hooks/                 # Shared hooks
│   │   ├── use-form.ts
│   │   ├── use-debounce.ts
│   │   ├── use-network-status.ts
│   │   └── index.ts
│   ├── utils/                 # Utility functions
│   │   ├── formatters.ts
│   │   ├── validators.ts
│   │   └── index.ts
│   ├── types/                 # Shared types
│   │   ├── api.ts
│   │   └── index.ts
│   └── constants/             # Shared constants
│       ├── colors.ts
│       └── index.ts
│
├── store/                     # Global state (Zustand)
│   ├── auth-store.ts          # Auth state
│   ├── theme-store.ts         # Theme state
│   ├── network-store.ts       # Network/online status
│   ├── sync-store.ts          # Sync queue status
│   └── index.ts               # Store exports
│
└── __tests__/                 # Global test utilities
    ├── setup.ts               # Jest setup
    ├── mocks/                 # Global mocks
    │   ├── api.ts
    │   ├── navigation.ts
    │   └── stores.ts
    └── utils/                 # Test utilities
        ├── render.tsx         # Custom render with providers
        └── factories.ts       # Test data factories

locales/                       # i18n translation files
├── en/
│   ├── common.json            # Common translations
│   ├── auth.json              # Auth feature translations
│   ├── settings.json          # Settings translations
│   └── errors.json            # Error messages
└── zh-TW/
    ├── common.json
    ├── auth.json
    ├── settings.json
    └── errors.json
```

## Architecture Layers

| Layer | Purpose | Dependencies | Example |
|-------|---------|--------------|---------|
| `app/` | Screens & navigation (Expo Router) | All layers | `app/(tabs)/index.tsx` |
| `features/` | Self-contained feature modules | `shared/`, `core/`, `store/` | Auth, transactions, settings |
| `shared/` | Cross-feature reusable code | `core/` only | UI components, hooks, utils |
| `core/` | Infrastructure & services | None (base layer) | API client, i18n, providers |
| `store/` | Global state management | `core/`, `shared/` | Auth state, theme, network |

## Dependency Rules

```
┌─────────────────────────────────────────────────────────┐
│                         app/                             │
│                    (screens, navigation)                 │
└─────────────────────────┬───────────────────────────────┘
                          │ imports from
                          ▼
┌─────────────────────────────────────────────────────────┐
│                      features/                           │
│              (self-contained modules)                    │
│                                                          │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐               │
│  │  auth   │   │ trans.  │   │settings │  ← NO cross   │
│  └────┬────┘   └────┬────┘   └────┬────┘    imports!   │
└───────┼─────────────┼─────────────┼─────────────────────┘
        │             │             │
        └─────────────┼─────────────┘
                      │ imports from
                      ▼
┌─────────────────────────────────────────────────────────┐
│                       shared/                            │
│           (cross-feature reusable code)                  │
└─────────────────────────┬───────────────────────────────┘
                          │ imports from
                          ▼
┌─────────────────────────────────────────────────────────┐
│                        core/                             │
│              (infrastructure, services)                  │
│                   NO DEPENDENCIES                        │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                        store/                            │
│              (global state - Zustand)                    │
│         Accessible from all layers above                 │
└─────────────────────────────────────────────────────────┘
```

### Rules

1. **`core/`** - Base layer, no dependencies on other layers
2. **`shared/`** - Only imports from `core/`
3. **`features/`** - Imports from `shared/` and `core/`, **never from other features**
4. **`store/`** - Imports from `core/` and `shared/`, accessible by all layers
5. **`app/`** - Can import from all layers

### Cross-Feature Communication

```typescript
// ❌ WRONG: Direct cross-feature import
import { useAuth } from '@/features/auth';

// ✅ CORRECT: Use global store for shared state
import { useAuthStore } from '@/store';

// ✅ CORRECT: Use events/callbacks through navigation
router.push('/login', { onSuccess: handleAuthSuccess });
```

## Feature Module Structure

Each feature should be self-contained with this structure:

```
features/[feature-name]/
├── api/               # React Query hooks for this feature
│   ├── use-[resource].ts        # Query hook
│   ├── use-create-[resource].ts # Mutation hook
│   └── index.ts
│
├── components/        # Feature-specific components
│   ├── screens/       # Screen-level components (used in app/)
│   │   └── [feature]-screen.tsx
│   └── widgets/       # Reusable within feature
│       └── [widget-name].tsx
│
├── hooks/             # Feature-specific hooks
│   └── use-[hook-name].ts
│
├── store/             # Feature-local state (optional)
│   └── [feature]-store.ts
│
├── schemas/           # Zod validation schemas
│   └── [schema-name].ts
│
├── types/             # Feature types
│   └── index.ts
│
├── utils/             # Feature utilities (if needed)
│   └── [util-name].ts
│
├── __tests__/         # Feature tests
│   ├── components/
│   ├── hooks/
│   └── api/
│
└── index.ts           # Public API (barrel export)
```

### Barrel Export Pattern

```typescript
// features/auth/index.ts
// Only export public API
export * from './api';
export * from './components/screens';
export * from './hooks';
export type * from './types';

// DO NOT export internal widgets, store, utils
```

## Import Aliases

Configure `@/` alias in `tsconfig.json` (use `--import-alias` when creating project):

```typescript
// Use @/ for src/ directory
import { Button } from '@/shared/components/ui';
import { useAuth } from '@/features/auth';
import { client } from '@/core/services/api';
import { useAuthStore } from '@/store';
import { useTranslation } from '@/core/i18n';
```

## Expo Router Conventions

### Route Groups

- `(tabs)/` - Tab navigation
- `(auth)/` - Authentication flow (unauthenticated)
- `(app)/` - Main app (authenticated)

### Special Files

- `_layout.tsx` - Layout wrapper for route segment
- `[param].tsx` - Dynamic route
- `[...catchAll].tsx` - Catch-all route
- `+not-found.tsx` - 404 page

## i18n Structure

Mixed namespace approach for translations:

```
locales/
├── en/
│   ├── common.json      # Shared: buttons, navigation, common words
│   ├── errors.json      # Error messages
│   ├── auth.json        # Auth feature strings
│   ├── settings.json    # Settings feature strings
│   └── transactions.json
└── zh-TW/
    └── ...
```

See [references/i18n.md](./i18n.md) for detailed configuration.
